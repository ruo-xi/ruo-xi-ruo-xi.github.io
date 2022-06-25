# Jdk Concurrency

# Concurrency

## conception

* concurrent(concurrency)

  when two or more tasks can start, run, and complete in overlapping time periods. It doesn't necessarily mean they'll ever both be running at the same instant

* parallel(parallelism)

  when tasks literally run at the same time, e.g., on a multicore processor.

cache line size  /proc/cpuinfo   cache_alignment	: 64 byte

## Feature
  bug source visibility, orderliness, atomicity

### visibility

* volatile 
* memory fence
* synchronized
* Lock
* final

###  orderliness

* volatile
* memory fence
* synchronized
* Lock

###  atomicity

* synchronized
* Lock
* CAS


## fundamental principle

### volatile

#### feature

* visibility

  each writing of a variable decorated by volatile can be awared for each thread

  对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

* atomicity

  read/write is a atomic op for variable decorated by volatile 

  对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性（基于这点，我们通过会认为volatile不具备原子性）。volatile仅仅保证对单个volatile变量的读/写具有原子性，而锁的互斥执行的特性可以确保对整个临界区代码的执行具有原子性。 
  > 64位的long型和double型变量，只要它是volatile变量，对该变量的读/写就具有原子性。

* orderliness

  对volatile修饰的变量的读写操作前后加上各种特定的内存屏障来禁止指令重排序来保障有序性。 

#### source

##### bytecode
```c
// src/hotspot/share/interpreter/zero/bytecodeInterpreter.cpp

CASE(_putfield):
CASE(_putstatic):
{
  ...

  if (cache->is_volatile()) {

    ...

    OrderAccess::storeload();
  } 
  
  UPDATE_PC_AND_TOS_AND_CONTINUE(3, count);
}

```

```c
// src/hotspot/share/interpreter/zero/bytecodeInterpreter.cpp

...

#include "runtime/orderAccess.hpp"

...

```

```c
// src/hotspot/share/runtime/orderAccess.hpp 

class OrderAccess : public AllStatic {
 public:
  // barriers
  static void     loadload();
  static void     storestore();
  static void     loadstore();
  static void     storeload();

  static void     acquire();
  static void     release();
  static void     fence();

  ...

}

```

```c

#include OS_CPU_HEADER(orderAccess)

// src/hotspot/os_cpu/linux_x86/orderAccess_linux_x86.hpp

inline void OrderAccess::loadload()   { compiler_barrier(); }
inline void OrderAccess::storestore() { compiler_barrier(); }
inline void OrderAccess::loadstore()  { compiler_barrier(); }
inline void OrderAccess::storeload()  { fence();            }

inline void OrderAccess::acquire()    { compiler_barrier(); }
inline void OrderAccess::release()    { compiler_barrier(); }

inline void OrderAccess::fence() {
// always use locked addl since mfence is sometimes expensive
#ifdef AMD64
  __asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");
#else
  __asm__ volatile ("lock; addl $0,0(%%esp)" : : : "cc", "memory");
#endif
  compiler_barrier();
}

```

##### template

```c
// src/hotspot/share/interpreter/templateInterpreter.cpp 

void TemplateInterpreter::initialize_code() {
  AbstractInterpreter::initialize();

  TemplateTable::initialize();

  ...

}

```

```c
// src/hotspot/share/interpreter/templateTable.cpp

void TemplateTable::initialize() {

  ...

  def(Bytecodes::_getstatic           , ubcp|____|clvm|____, vtos, vtos, getstatic           , f1_byte      );
  def(Bytecodes::_putstatic           , ubcp|____|clvm|____, vtos, vtos, putstatic           , f2_byte      );
  def(Bytecodes::_getfield            , ubcp|____|clvm|____, vtos, vtos, getfield            , f1_byte      );
  def(Bytecodes::_putfield            , ubcp|____|clvm|____, vtos, vtos, putfield            , f2_byte      );

  ...
}
```

```c
// src/hotspot/cpu/x86/templateTable_x86.cpp


void TemplateTable::putfield(int byte_no) {
  putfield_or_static(byte_no, false);
}


void TemplateTable::putstatic(int byte_no) {
  putfield_or_static(byte_no, true);
}


void TemplateTable::putfield_or_static(int byte_no, bool is_static, RewriteControl rc) {
  transition(vtos, vtos);

  const Register cache = rcx;
  const Register index = rdx;
  const Register obj   = rcx;
  const Register off   = rbx;
  const Register flags = rax;

  resolve_cache_and_index(byte_no, cache, index, sizeof(u2));
  jvmti_post_field_mod(cache, index, is_static);
  load_field_cp_cache_entry(obj, cache, index, off, flags, is_static);

  // [jk] not needed currently
  // volatile_barrier(Assembler::Membar_mask_bits(Assembler::LoadStore |
  //                                              Assembler::StoreStore));

  Label notVolatile, Done;
  __ movl(rdx, flags);
  __ shrl(rdx, ConstantPoolCacheEntry::is_volatile_shift);
  __ andl(rdx, 0x1);

  // Check for volatile store
  __ testl(rdx, rdx);
  __ jcc(Assembler::zero, notVolatile);

  putfield_or_static_helper(byte_no, is_static, rc, obj, off, flags);
  volatile_barrier(Assembler::Membar_mask_bits(Assembler::StoreLoad |
                                               Assembler::StoreStore));
  __ jmp(Done);
  __ bind(notVolatile);

  putfield_or_static_helper(byte_no, is_static, rc, obj, off, flags);

  __ bind(Done);
}

void TemplateTable::volatile_barrier(Assembler::Membar_mask_bits order_constraint ) {
  // Helper function to insert a is-volatile test and memory barrier
  __ membar(order_constraint);
}

void Assembler::membar(Membar_mask_bits order_constraint) {
  // We only have to handle StoreLoad
  if (order_constraint & StoreLoad) {
    // All usable chips support "locked" instructions which suffice
    // as barriers, and are much faster than the alternative of
    // using cpuid instruction. We use here a locked add [esp-C],0.
    // This is conveniently otherwise a no-op except for blowing
    // flags, and introducing a false dependency on target memory
    // location. We can't do anything with flags, but we can avoid
    // memory dependencies in the current method by locked-adding
    // somewhere else on the stack. Doing [esp+C] will collide with
    // something on stack in current method, hence we go for [esp-C].
    // It is convenient since it is almost always in data cache, for
    // any small C.  We need to step back from SP to avoid data
    // dependencies with other things on below SP (callee-saves, for
    // example). Without a clear way to figure out the minimal safe
    // distance from SP, it makes sense to step back the complete
    // cache line, as this will also avoid possible second-order effects
    // with locked ops against the cache line. Our choice of offset
    // is bounded by x86 operand encoding, which should stay within
    // [-128; +127] to have the 8-byte displacement encoding.
    //
    // Any change to this code may need to revisit other places in
    // the code where this idiom is used, in particular the
    // orderAccess code.

    int offset = -VM_Version::L1_line_size();
    if (offset < -128) {
      offset = -128;
    }

    lock();
    addl(Address(rsp, offset), 0);// Assert the lock# signal here
  }
}

```

###### confirm

  confirm by add **-Xcomp** **-XX:+UnlockDiagnosticVMOptions** **-XX:+PrintAssembly** vm options to output assemble code, and you need to add **hsdis-amd64.so**  to **{JDK11_HOME}/lib/** 

  ``` bash
    java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -Xcomp src/main/java/top/laonaailifa/jdk/concurrent/test/Visibility.java > output_file
    grep "putfield flag " --after-context=2 output_file

    output: 
      4140204:  0x00007f4e49557cad: lock addl $0x0,-0x40(%rsp)  ;*putfield flag {reexecute=0 rethrow=0 return_oop=0}
      4140205-                           ; - top.laonaailifa.jdk.concurrent.test.Visibility::<init>@6 (line 7)
      4140206-                           ; - top.laonaailifa.jdk.concurrent.test.Visibility::main@5 (line 59)
  ```


## reference

* [What is the difference between concurrency and parallelism?](https://stackoverflow.com/questions/1050222/what-is-the-difference-between-concurrency-and-parallelism)


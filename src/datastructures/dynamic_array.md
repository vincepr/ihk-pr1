# Basic structures
The basic struct most other structs are made out of
## Static Array vs Dynamic Array
### Static Array
Continous chunk of memory. To reserve/allocate the memory the size has to be known at time of creation.
- This means they can be allocated on the stack (if whatever we store is fixed size). So really fast
- Modern CPU's are also really good at Cashing, so consecutive access to the same array can be really fast that way.
### Dynamic Array
Extension of a Statc Array but, gets reallocated once full (or if it shrinks enough).

|||||
|---|---|---|---|
|Access | O(1) | O(1) |can index directly into|
|Search | O(n) | O(n) |worst case whole array gets run trough|
|Insertion| N/A | O(n) |worst case we insert at idx=0 so we have to shift everything right|
|Appending| N/A | O(1) |really fast to `push()` or `pop()`|
|Deletion | N/A | O(n) |worst case we delete idx=0 so we have to shift everything left|

## Implementation Dynamic Array in C
First we take care of memory allocation with some generic macros:
- `memory.h`
```c
#ifndef custom_memory_h
#define custom_memory_h

// macro that allocates an array with given type and count/size:
#define ALLOCATE(type, count) \
    (type*)reallocate(NULL, 0, sizeof(type) * (count))

// macro - used to free memory that a custom Object used
#define FREE(type, pointer) reallocate(pointer, sizeof(type), 0)

// We define we double Capacity whenever we reach its limit
// - we start at 8 (after upgrade from when it gets initialized with capacity = 0)
// - afterwards we double each time 8 -> 16 -> 32 -> 64 -> 128....
#define GROW_CAPACITY(capacity) \
    ((capacity) < 8 ? 8 : (capacity) * 2)

// This macro calls reallocate with newSize=0 -> so it will get deleted from memory there
#define FREE_ARRAY(type, pointer, oldCount) \
    reallocate(pointer, sizeof(type) * (oldCount), 0);

void* reallocate(void* pointer, size_t oldSize, size_t newSize);
#endif
```
- `memory.c`
```c
#include <stdlib.h>
#include "memory.h"

//  The single function used for all dynamic memory management
    // if   -oldSize-   -newSize-       -then do Operation:-
    //      0           Non-zero        Allocate new block-
    //      Non-Zero    0               Free allocation.
    //      Non-Zero    new<oldSize     Shrink existing allocation.
    //      Non-Zero    new>oldSize     Grow existing allocation.
void* reallocate(void* pointer, size_t oldSize, size_t newSize) {
    if (newSize == 0) {
        free(pointer);
        return NULL;
    }

    void* result = realloc(pointer, newSize);
    if (result == NULL) exit(1);                // We must handle the case of realloc failing (ex. not enough free memory left on OS)
    return result;
}
```
- `array.h`
```c
#ifndef custom_array_h
#define custom_array_h

typedef struct {
    int capacity;
    int count;
    Int* values;    // pointer to our static-array
} IntArray;

void initIntArray(IntArray* array);
void pushToArray(IntArray* array, int value);
void deleteFromArray(IntArray* array, int index);
void freeIntArray(IntArray* array);


#endif
```
- `array.c`
```c
#include <stdio.h>
#include "memory.h"
#include "varray.h"

// inititializing the dynamic data structure
void initIntArray(IntArray* array) {
    array->values = NULL;
    array->capacity = 0;
    array->count = 0;
}

// appends new value to array, if full it reallocates to double'd size
void pushToArray(IntArray* array, int value) {
    if (array->capacity < array->count + 1) {
        int oldCapacity = array->capacity;
        array->capacity = GROW_CAPACITY(oldCapacity);
        array->values = GROW_ARRAY(Value, array->values, oldCapacity, array->capacity);
    }
    array->values[array->count] = value;
    array->count++;
}

// deletes at idx specified
void deleteFromArray(IntArray* array, int index) {
    for (int i = index; i < array->count -1; i++) {
        array->items[i] = array->items[i+1];
    }
    array->items[array->count-1] = NULL;    // cleanup the memory were not using anymore
    array->count--;
}

// after done we have to manually malloc the memory
void freeIntArray(IntArray* array) {
    FREE_ARRAY(Int, array->values, array->capacity);
    initIntArray(array);
}

void arrayWriteToIdx(IntArray* array, int index, int value) {
    array->items[index] = value;
}

int arrayReadFromIdx() {
    return array->values[index]
}

bool isValidIndex(IntArray* array, int index) {
    return (index >= 0 && index < array->count)
}

int arrayGetLength(IntArray* array) {
    return array->count;
}
```

## Implementation Dynamic Array in Rust

Rustnomicon-Book on the full unsafe guide for rust. Definitly gotta read that book when i have the time.

```rust
use std::{mem, ptr::NonNull, alloc::Layout};

fn main() {
    let mut arr:Array<&str> = Array::new();
    arr.push("Hello");
    arr.push("World");
    arr.push("1");
    arr.push("16");
    dbg!(&arr[0]);
    dbg!(&arr[1]);
    dbg!(&arr[3]);


}

#[derive(Debug)]
/// dynamic Array implementation - of note: Vec is already mostly what were building, so this is more a theoretical practice thing
pub struct Array<T> {
    buf: RawArray<T>,
    len: usize,
}

impl<T> Array<T> {
    /// just forwards our pointer from our RawArray
    fn ptr(&self) -> *mut T {
        self.buf.ptr.as_ptr()
    }

    /// just forwards our capacity from our RawArray
    fn cap(&self) -> usize {
        self.buf.cap
    }
    
    /// initializes the Array struct. BUT first allocation on the heap has not happened. (only when filled)
    pub fn new() -> Self {
        Array {
           buf: RawArray::new(),
            len: 0,
        }
    }

    /// pushes the element on top of the Array.
    pub fn push(&mut self, elem: T) {
        if self.len == self.cap() {self.buf.grow(); }
        unsafe {
            std::ptr::write(self.ptr().add(self.len), elem);
        }
        self.len += 1;
    }

    /// pops top element from Array, if it exists.
    pub fn pop(&mut self) -> Option<T> {
        if self.len == 0 {
            None
        } else {
            self.len -=1;
            unsafe {
                // ptr::read() copies the bits from adress and interprets it as T. 
                // we offset by self.len to get to the top of the stack
                Some(std::ptr::read(self.ptr().add(self.len)))
            }
        }
    }

    /// insert and shift to right if not inserting at top
    /// - we use ptr::copy == C's memmove -> copies a chunk of memory 
    pub fn insert(&mut self, index: usize, elem: T) {
        assert!(index <= self.len, "index is out of bounds.");
        if self.cap() == self.len { self.buf.grow(); }
        unsafe {
            // first we copy everything at the idx one idx to the right
            std::ptr::copy(
                self.ptr().add(index),
                self.ptr().add(index - 1),
                self.len - index,
            );
            // then we insert our new element at the index
            std::ptr::write(self.ptr().add(index), elem);
            self.len += 1;
        }
    }

    /// removes one element and shifts to left if there was something above it
    /// - we use ptr::copy == C's memmove -> copies a chunk of memory 
    pub fn remove(&mut self, index: usize) -> T {
        assert!(index < self.len, "index is out of bounds.");
        unsafe {
            self.len -=1;
            let result = std::ptr::read(
                self.ptr().add(index));
            std::ptr::copy(
                self.ptr().add(index +1),
                self.ptr().add(index),
                self.len - index,
            );
            return result;
        }
    }
}

impl <T> Drop for Array<T> {
    /// Drop trait - destructor that cleans up . we just do pop() till empty
    /// - Drop implementation for RawArray handles the Deallocation
    fn drop(&mut self) {
        while let Some(_) = self.pop() {}   // loop till empty
    }
}

// Deref to enable []- Syntax to rea values in the array. ex: 'dbg!(arr[1]);'
impl<T> std::ops::Deref for Array<T> {
    type Target = [T];
    fn deref(&self) -> &[T] {
        unsafe {
            std::slice::from_raw_parts(self.ptr(), self.len)
        }
    }
}
// Deref to enable []- Syntax to write to values in the array. ex: 'arr[1]="hello";'
impl<T> std::ops::DerefMut for Array<T> {
    fn deref_mut(&mut self) -> &mut [T] {
        unsafe {
            std::slice::from_raw_parts_mut(self.ptr(), self.len)
        }
    }
}





// Note: 'iter' and 'iter_mut' we got automagically with Deref. But 'into_iter' and 'drain' not.
// - (those consume the Array by-value and yield its elements value by value)
// they get implemented as DoubleEnded Iterator -> with 2 pointers pointing to the ends.
// - 'end' points AFTER the element it wants read next. 'start' points DIRECTLY at the element. (for clarity when done)
pub struct IntoIter<T> {
    _buf: RawArray<T>,
    start: *const T,
    end: *const T,
}

// because IntoIter takes ownership it NEEDS to Drop to free it. (ex. elements that were not yielded)
impl<T> Drop for IntoIter<T> {
    fn drop(&mut self) {
        // only need to ensure all our elements get droped, RawArray buffer will clean up after itself
        for _ in &mut *self {}
    }
}

impl<T> IntoIterator for Array<T> {
    type Item = T;
    type IntoIter = IntoIter<T>;

    fn into_iter(self) -> IntoIter<T> {
        unsafe {
            // need to use ptr::read to unsafely move the buf out (and we cant destructure it)
            let buf = std::ptr::read(&self.buf);
            let len = self.len;
            mem::forget(self);


            IntoIter {
                start: buf.ptr.as_ptr(),
                end: if buf.cap ==0 {
                    buf.ptr.as_ptr()    // we can only offset/add off the pointer if something is allocated
                } else {
                    buf.ptr.as_ptr().add(len)
                },
                _buf: buf,
            }
        }
    }

}

impl<T> Iterator for IntoIter<T> {
    type Item = T;
    /// consumes next element in array - read out what next points to then increment next.
    fn next(&mut self) -> Option<T> {
        if self.start == self.end {
            None    // iterator is empty
        } else {
            unsafe {
                let result = std::ptr::read(self.start);
                self.start = self.start.offset(1);
                Some(result)
            }
        }
    }

    fn size_hint(&self) -> (usize, Option<usize>) {
        let len = (self.end as usize - self.start as usize) / mem::size_of::<T>();
        (len, Some(len))
    }
}

impl<T> DoubleEndedIterator for IntoIter<T> {
    /// consumes last element from array - end points to after what we want to read -> so we offset first then read 
    fn next_back(&mut self) -> Option<Self::Item> {
        if self.start == self.end {
            None    // iterator is empty
        } else {
            unsafe {
                self.end = self.end.offset(-1);
                let result = std::ptr::read(self.end);
                Some (result)
            }
        }
    }
}

// the pointer (and capacity) get abstracted out from Array.
// - this way all the unsafe memory allocation gets decoupled/gathered in one spot.
#[derive(Debug)]
struct RawArray<T> {
    ptr: NonNull<T>,
    cap: usize,
}
// by implementing Send/Sync ontop of the NonNull<T> pointer we get the same benefits of Unique<T>

unsafe impl <T: Send> Send for RawArray<T> {}
unsafe impl <T: Sync> Sync for RawArray<T> {}

impl<T> RawArray<T> {
    fn new() -> Self {
        assert!(mem::size_of::<T>() !=0, "TODO: implement Zero size allocation");
        RawArray { 
            ptr: NonNull::dangling(), // like Unique<T> a wrapper over a raw pointer. (that cant be NULL and must be of type T)
            cap: 0, 
        }
    }
    /// grow will take care of all the memory allocation happening
    /// - will double size once capacity is reached
    /// - if out of memory-error it will call system implemention to abort
    fn grow(&mut self) {
        const INITIALCAP:usize = 8;     // first time we grow above initial len=cap=0 we skipp to this.
        let (new_cap, new_layout) = if self.cap==0 {
            (INITIALCAP, Layout::array::<T>(INITIALCAP).unwrap()) // since len=cap=0 we need to start with some bigger value
        } else {
            let new_cap = 2 * self.cap;
            let new_layout = Layout::array::<T>(new_cap).unwrap();
            (new_cap, new_layout)
        };
        // ensure that the new allocation is inside what a 32 or 64bit system pointer can handle
        // staying inside those sizes and the above Layouts shouldn never unwrap
        assert!(new_layout.size() <= isize::MAX as usize, "Allocation to large");

        // depending if cap was 0 previous we first allocate or reallocate memory with bigger cap:
        let new_ptr = if self.cap == 0 {
            unsafe { std::alloc::alloc(new_layout) }
        } else {
            let old_layout = Layout::array::<T>(self.cap).unwrap();
            let old_ptr = self.ptr.as_ptr() as * mut u8;
            unsafe { std::alloc::realloc(old_ptr, old_layout, new_layout.size()) }
        };

        // if allocation fails (not enough free memory) new_ptr will be null -> we abort (not just exit)
        self.ptr = match NonNull::new(new_ptr as *mut T) {
            Some(p) =>p,
            None => std::alloc::handle_alloc_error(new_layout), // calls system default no-memory-abort
        };
        self.cap = new_cap;
    }
}

impl <T> Drop for RawArray<T> {
    /// Drop trait - destructor that cleans up -> in this case deallocates our memory
    /// - if self.cap == 0 no allocation happened and we skip deallocation
    fn drop(&mut self) {
        if self.cap != 0 {
            let layout = Layout::array::<T>(self.cap).unwrap();
            unsafe{
                std::alloc::dealloc(self.ptr.as_ptr() as *mut u8, layout);
            }
        }
    }
}

```
#### Halide Basic

- `Func` : a pipeline stage
- `Var`: use as variables in the definition of `Func`

- `Expr`: computing of variables

```C++
// definition of func
Halide::Var x, y;
Halide::Func gradient(x, y) = x + y; // notice: x is column index, y is row index
// row-major traversal
// meta-programming
// computing
Halide::Buffer<int32_t> output = gradient.realize(800, 600);
```

##### Debugging methods

- `gradient.gradient.trace_stores()`
- gradient.gradient.print_loop_nest();

- `gradient.compile_to_lowered_stmt("gradient.html", {}, HTML);` print pesudo code generated from halide



#### Scheduler

- reorder

  `  gradient.reorder(y, x);` reorder the loop order

- split

  ```c++
  gradient.split(x, x_outer, x_inner, 2);
  ```

- fuse

  ```c++
   Var fused;
   gradient.fuse(x, y, fused);
  ```

- tile

  ```c++
  Var x_outer, x_inner, y_outer, y_inner;
  gradient.split(x, x_outer, x_inner, 4);
  gradient.split(y, y_outer, y_inner, 4);
  gradient.reorder(x_inner, y_inner, x_outer, y_outer);
  
  // equals to:
  // gradient.tile(x, y, x_outer, y_outer, x_inner, y_inner, 4, 4);
  ```

  ![img](lesson_05_tiled.gif)

- vectors

  ```c++
  Var x_outer, x_inner;
  gradient.split(x, x_outer, x_inner, 4);
  gradient.vectorize(x_inner);
  
  // equal to:
  // gradient.vectorize(x, 4);
  ```

- Unrolling a loop

  ```c++
  Var x_outer, x_inner;
  gradient.split(x, x_outer, x_inner, 2);
  gradient.unroll(x_inner);
  
  // The shorthand for this is:
  // gradient.unroll(x, 2);
  
  //(similar to vectorize, but not sychronously computing the values in the unrolled dimension
  ```

- splittin factors dont divide by extent

  ```c++
  // The general rule is: If we require x from x_min to x_min + x_extent, and
  // we split by a factor 'factor', then:
  //
  // x_outer runs from 0 to (x_extent + factor - 1)/factor
  // x_inner runs from 0 to factor
  // x = min(x_outer * factor, x_extent - factor) + x_inner + x_min
  ```

- Tile parallelizing

  ```c++
  Var x_outer, y_outer, x_inner, y_inner, tile_index;
  gradient.tile(x, y, x_outer, y_outer, x_inner, y_inner, 4, 4);
  gradient.fuse(x_outer, y_outer, tile_index);
  gradient.parallel(tile_index);
  ```

#### Scheduling between Funcs(multistage)

consider the producer and consumer example on the official website:

```c++
producer(x, y) = sin(x * y);

// Now we'll add a second stage which averages together multiple
// points in the first stage.
consumer(x, y) = (producer(x, y) +
                  producer(x, y+1) +
                  producer(x+1, y) +
                  producer(x+1, y+1))/4;
```

1) fully inline(directly compile on the original code)

No stores and loads happens, fully usage of cpu to compute `sin`

2) fully stores at memory

` producer.compute_root();`

stores all producer results in memory

3) trade off between 1 and 2

`producer.compute_at(consumer, y)`

To evaluate producer as needed per y(row) coordinate of the consumer.

4) more memory efficiency

```c++
producer.compute_root().compute_at(consumer, y);
```

rather than allocating $5\times5$ temporary memory for producer array, the Halide allocates $2\times5$ memory and wrap the index by 2 in the optimization.

```c++
int producer[2][5];
int consumer[4][4];
for(int y=0; y<4; y++){
	for(int yp=y; yp<y+2; yp++){
		for(int yx=0;yx<5;yx++){
			if(yp > 0 && yp == y) continue;
            producer[yp & 1][yx] = sin(yp * xp);
            // the even row number of consumer allocate at 0 row of producer
            // the odd number of consumer allocate at 1st row of producer
		}
	}
    for(int x = 0; x < 4; x++) {  
    	//consumer...
        consumer[x][y] = (producer[y & 1][x] + producer[y & 1][x + 1] + 
                          producer[(y + 1) & 1][x] + producer[(y+1) & 1][x+1]
        ) / 4;
    }
}
```

Therefore, the temporary memory allocation size: 10 units, loads memory operation times: $4\times16$, stores memory operation times: $4\times4+5\times5$

5) doing best

Combines 

```
// Store outermost, compute innermost.
producer.store_root().compute_at(consumer, x);
```

#### Scheduling Update step

definition of update and pure definition

```c++
// For each realization of f, each step runs in its entirety
// before the next one begins. Let's trace the loads and
// stores for a simpler example:
Func g("g");
g(x, y) = x + y;   // Pure definition
g(2, 1) = 42;      // First update definition
g(x, 0) = g(x, 1); // Second update definition
```

Scheduling update steps

```c++
// Consider the definition:
Func f;
f(x, y) = x * y;
// Set row zero to each row 8
f(x, 0) = f(x, 8);
// Set column zero equal to column 8 plus 2
f(0, y) = f(8, y) + 2;

// The pure variables in each stage can be scheduled
// independently. To control the pure definition, we schedule
// as we have done in the past. The following code vectorizes
// and parallelizes the pure definition only.
f.vectorize(x, 4).parallel(y);

// We use Func::update(int) to get a handle to an update step
// for the purposes of scheduling. The following line
// vectorizes the first update step across x. We can't do
// anything with y for this update step, because it doesn't
// use y.
f.update(0).vectorize(x, 4);

// Now we parallelize the second update step in chunks of size
// 4.
Var yo, yi;
f.update(1).split(y, yo, yi, 4).parallel(yo);
```



#### Environments Variables:

`HL_DEBUG_CODEGEN` debug switch

` HL_NUM_THREADS` control thread num
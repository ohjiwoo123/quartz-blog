https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#c-language-extensions

`__global__` : CPU에 의해서 실행되는 GPU 커널 함수이다.
(무조건 VOID를 RETURN 한다.)

`__device__` : GPU 함수내에서 호출이 가능하다 

`__host__`: CPU 함수 내에서 호출이 가능하다 (default)


## allocate device(GPU) memory

```
int main() 
{

float *d_x;
float *d_y;

cudaMaloc((void**) &d_x, n*sizeof(float));
cudaMaloc((void**) &d_x, n*sizeof(float));

// copy x and y from host memory to device memory
cudaMemcpy(d_x, x, n*sizeof(float), cudaMemcpyHostToDevice);
cudaMemcpy(d_y, y, n*sizeof(float), cudaMemcpyHostToDevice);

// invoke parallel SAXPY kernel with 256 threads / block
int nblocks = (n+255)/256
saxpy<<<nblocks, 256>>>(n, 2.0, d_x, d_y);

// do something with the result...

// free device (GPU) memory
cudaFree(d_x);
cudaFree(d_y);

}

return 0;

```
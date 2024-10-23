As a thought with this bit:

```
   **Original flags:**
   `-Wno-parentheses-equality -Wno-unused-value -ffast-math`
   
   **Modified flags:**
   `-ffast-math -Wno-parentheses-equality -Wno-unused-value -mllvm -amdgpu-early-inline-all=true -mllvm -amdgpu-function-calls=false -O3 --offload-device-only -fno-math-errno -fno-signed-zeros -fno-trapping-math -ffp-contract=fast`
```

Are you *sure* `-ffast-math` is a good idea? :smile:

It has a terrible reputation for generating code which will output incorrect results in some situations.

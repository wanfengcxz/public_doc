

```python
import torch
import torchvision.models as models
from torch.profiler import profile, record_function, ProfilerActivity

if __name__ == "__main__":
    model = models.resnet18()
    inputs = torch.randn(5, 3, 224, 224)
    
    with profile(activities=[ProfilerActivity.CPU], 
                 record_shapes=True,
                 profile_memory=True) as prof:
        with record_function("model_inference"):
            model(inputs)
            
    # print(prof.key_averages().table(sort_by="cpu_time_total", row_limit=10))
    # print(prof.key_averages(group_by_input_shape=True).table(sort_by="cpu_time_total", row_limit=10))
    # print(prof.key_averages().table(sort_by="self_cpu_memory_usage", row_limit=10))
    prof.export_chrome_trace("trace.json")
```

输出

```bash
STAGE:2024-09-04 18:43:09 17203:17203 ActivityProfilerController.cpp:312] Completed Stage: Warm Up
STAGE:2024-09-04 18:43:09 17203:17203 ActivityProfilerController.cpp:318] Completed Stage: Collection
STAGE:2024-09-04 18:43:09 17203:17203 ActivityProfilerController.cpp:322] Completed Stage: Post Processing
---------------------------------  ------------  ------------  ------------  ------------  ------------  ------------  ------------  ------------  
                             Name    Self CPU %      Self CPU   CPU total %     CPU total  CPU time avg       CPU Mem  Self CPU Mem    # of Calls  
---------------------------------  ------------  ------------  ------------  ------------  ------------  ------------  ------------  ------------  
                  model_inference         3.60%       9.001ms       100.00%     250.185ms     250.185ms           0 b    -106.30 Mb             1  
                     aten::conv2d         0.78%       1.963ms        62.16%     155.526ms       7.776ms      47.37 Mb           0 b            20  
                aten::convolution         0.35%     864.000us        61.38%     153.563ms       7.678ms      47.37 Mb           0 b            20  
               aten::_convolution         0.45%       1.122ms        61.03%     152.699ms       7.635ms      47.37 Mb           0 b            20  
         aten::mkldnn_convolution        60.25%     150.728ms        60.59%     151.577ms       7.579ms      47.37 Mb           0 b            20  
                 aten::max_pool2d         0.00%       9.000us        15.00%      37.521ms      37.521ms      11.48 Mb           0 b             1  
    aten::max_pool2d_with_indices        14.99%      37.512ms        14.99%      37.512ms      37.512ms      11.48 Mb      11.48 Mb             1  
                 aten::batch_norm        -0.15%    -368.000us         8.92%      22.326ms       1.116ms      47.41 Mb    -982.00 Kb            20  
     aten::_batch_norm_impl_index         0.41%       1.034ms         8.91%      22.292ms       1.115ms      47.41 Mb     982.00 Kb            20  
          aten::native_batch_norm         8.63%      21.594ms         8.66%      21.659ms       1.083ms      47.41 Mb     -12.49 Mb            20  
---------------------------------  ------------  ------------  ------------  ------------  ------------  ------------  ------------  ------------  
Self CPU time total: 250.185ms
```


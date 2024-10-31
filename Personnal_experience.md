# v100训练不稳定问题
在build_nanogpt项目中我发现了v100训练不稳定问题
v100上没有bf16只能用fp16，而且mem 32g小只能用小batch 16 （wordsize 8,T 1024,0.5m clip gradient ），结果训练不稳定,norm 在lr warmup到1e4之后 surge;
## 我怀疑fp16更直接有问题，但没法验证，也可能是结合了batch 16导致
-->用h800 batch 16+ bf16 没问题
-->不用量化直接慢死，训练速度降低到1/4
step    56 | loss: 8.319144 | lr 4.7832e-05 | norm: 6.4308 | dt: 4875.88ms | tok/sec: 107526.94
step    57 | loss: 8.266062 | lr 4.8671e-05 | norm: 6.4732 | dt: 4865.72ms | tok/sec: 107751.38
tok降到107k/s, 8 v100 32g,正常可以到435k/s
-->用compile 会00M 
-->compile false, warmup_steps+2000,前期还行，lr到了1e4后再次变得不稳定,当时norm还是在7-20,看起来某些batch very unlucky
QQQ: how about adjust lr based on norm?
-->compile false, warmup_steps+2000，max_lr = 1e-4 is 1/6 of original: ==> 到了1400多步又出现loss变大，这时lr在6e5，norm8-12
--> compile false, warmup_steps 751 ,increase gradiant accumulate steps * 4 ==> still not working,loss surge to 18+ at 6000 steps
--> compile false, warmup_steps 751 ,increase gradiant accumulate steps * 2, lr=lr/norm when norm > 1==> 

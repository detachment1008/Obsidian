# SSAA

SSAA(Super-Sampling Anti-aliasing)，即超级采样抗锯齿

走样的原因是信号频率高，采样率低，但是采样率是固定的，我们就只能降低信号的频率，即每次采样，都对采样点做滤波，从而降低信号采样率。

步骤：以每个像素多 4 个采样点为例
1. 创建新的 super depth buffer 和 super frame buffer，用于维护采样点的深度缓冲和帧缓冲
2. 每次对像素采样时，都用像素内这四个采样点代替
3. 最终再把新的 super frame buffer 通过取平均的方式，转化为渲染用的 frame buffer 即可

难点：找到像素和采样点的对应关系
- 在原先的基础上，把宽和高乘以 2 即可

# MSAA
# 📖MultiPassDedup

### Efficient Deduplicate for Anime Video Frame Interpolation
> When performing frame interpolation on anime footage, conventional deduplication methods often rely on  identification, which has many drawbacks, such as losing background textures and failing to correctly handle multiple characters drawn with different cadences in the same scene.
> 
> Through observation and summarization of patterns in anime videos, we found that repeatedly updating the original frames provides an easier and more effective solution to these issues.
Therefore, we developed this project to implement this approach. Combined with the powerful GMFSS interpolation algorithm, we can achieve excellent results in most anime scenarios.

![result](assert/result.gif)

Online Colab demo for AFI-ForwardDeduplicate: [[Colab]](https://github.com/Q8sh2ing/AFI-ForwardDeduplicate-Colab/blob/main/forward_dedup_Colab.ipynb)

## 👀Demos Videos(BiliBili)
### [Jujutsu Kaisen S2 NCOP](https://www.bilibili.com/video/BV16W421N7s5/?share_source=copy_web&vd_source=8a8926eb0f1d5f0f1cab7529c8f51282)
### [Houseki no Kuni NCOP](https://www.bilibili.com/video/BV1py4y1A7qj/?share_source=copy_web&vd_source=8a8926eb0f1d5f0f1cab7529c8f51282)

## 🔧Installation
```bash
git clone https://github.com/routineLife1/AFI-ForwardDeduplicate.git
cd DRBA
pip3 install -r requirements.txt
```
download weights from [Google Drive](https://drive.google.com/file/d/1_PjZCKso1Gpiw4V68wL1CL5kNIAmsqog/view?usp=sharing) and unzip it, put them to ./weights/


The cupy package is included in the requirements, but its installation is optional. It is used to accelerate computation. If you encounter difficulties while installing this package, you can skip it.


## ⚡Usage 
- normalize the source video to 24000/1001 fps by following command using ffmpeg **(If the INPUT video framerate is around 23.976, skip this step.)**
  ```bash
  ffmpeg -i INPUT -crf 16 -r 24000/1001 -preset slow -c:v libx265 -x265-params profile=main10 -c:a copy OUTPUT
  ```
- open the video and check out it's max consistent deduplication counts, (3 -> on Three, 2 -> on Two, 0 -> AUTO) **(If the INPUT video framerate is around 23.976, skip this step.)**
- run the follwing command to finish interpolation
  (N_FORWARD = max_consistent_deduplication_counts - 1) **(Under the most circumstances, -nf 0 can automatically determine an appropriate n_forward value)**
  ```bash
  python infer.py -i [VIDEO] -o [VIDEO_OUTPUT] -nf [N_FORWARD] -t [TIMES] -m [MODEL_TYPE] -s -st 0.3 -scale [SCALE]
  # or use the following command to export video at any frame rate
  python infer.py -i [VIDEO] -o [VIDEO_OUTPUT] -nf [N_FORWARD] -fps [OUTPUT_FPS] -m [MODEL_TYPE] -s -st 0.3 -scale [SCALE]
  ```
  
 **example(smooth a 23.976fps video with on three and interpolate it to 60fps):**

  ```bash
  ffmpeg -i E:/Myvideo/01_src.mkv -crf 16 -r 24000/1001 -preset slow -c:v libx265 -x265-params profile=main10 -c:a copy E:/Myvideo/01.mkv

  python infer.py -i E:/MyVideo/01.mkv -o E:/MyVideo/out.mkv -nf 2 -fps 60 -m gmfss -s -st 0.3 -scale 1.0
  ```

**Full Usage**
```bash
Usage: python infer.py -i in_video -o out_video [options]...
       
  -h                   show this help
  -i input             input video path (absolute path of output video)
  -o output            output video path (absolute path of output video)
  -fps dst_fps         target frame rate (default=60)
  -s enable_scdet      enable scene change detection (default Enable)
  -st scdet_threshold  ssim scene detection threshold (default=0.3)
  -hw hwaccel          enable hardware acceleration encode (default Enable) (require nvidia graph card)
  -s scale             flow scale factor (default=1.0), generally use 1.0 with 1080P and 0.5 with 4K resolution
  -m model_type        model type (default=gmfss)
  -nf n_forward        max consistent deduplication counts (default=3)
```

- input accept absolute video file path. Example: E:/input.mp4
- output accept absolute video file path. Example: E:/output.mp4
- dst_fps = target interpolated video frame rate. Example: 60
- enable_scdet = enable scene change detection.
- scdet_threshold = scene change detection threshold. The larger the value, the more sensitive the detection.
- hwaccel = enable hardware acceleration during encoding output video.
- scale = flow scale factor. Decrease this value to reduce the computational difficulty of the model at higher resolutions. Generally, use 1.0 for 1080P and 0.5 for 4K resolution.
- model_type = model type. Currently, gmfss, rife and gimm is supported.
- n_forward = max consistent deduplication counts.


## todo list
- [x] **Efficiency optimization**
- [ ] **Explain why this method is effective and write a guidence on how to support other vfi algorithms**
- [x] **Implement any framerate support for ForwardDeduplicate (smooth interpolation method)**

## limitations
> The "n_forward" parameter acts like the number of times the algorithm performs Spatiotemporal TTA (Spatiotemporal Test Time Augmentation) operations.
> Performing too many TTA operations may further improve smoothness and interpolation performance but lead to blurriness.
> 
> This method will change the animation rhythm to a certain extent

## Projects that use AFI-ForwardDeduplicate
[SVFI(commercial software)](https://store.steampowered.com/app/1692080/SVFI/)

## 🤗 Acknowledgement

Thanks for [Q8sh2ing](https://github.com/Q8sh2ing) implement the Online Colab Demo.

## Reference
[SpatiotemporalResampling](https://github.com/hyw-dev/SpatiotemporalResampling) [GMFSS](https://github.com/98mxr/GMFSS_Fortuna) [Practical-RIFE](https://github.com/hzwer/Practical-RIFE) [GIMM-VFI](https://github.com/GSeanCDAT/GIMM-VFI)

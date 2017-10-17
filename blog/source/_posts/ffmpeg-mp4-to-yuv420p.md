---
title: 视频解码原理-MP4转YUV420P
date: 2017-10-10 12:14:51
categories: "音视频"
tags:
	- "ffmpeg"
---

<font size=4>

本文主要是采用ffmpeg对mp4文件进行视频解码



	#include <jni.h>
	#include <string>
	#include <android/log.h>

	extern "C" {
	//编码
	#include "libavcodec/avcodec.h"
	//封装格式处理
	#include "libavformat/avformat.h"
	//像素处理
	#include "libswscale/swscale.h"
	}
	#define LOGI(FORMAT, ...) __android_log_print(ANDROID_LOG_INFO,"jason",FORMAT,##__VA_ARGS__);
	#define LOGE(FORMAT, ...) __android_log_print(ANDROID_LOG_ERROR,"jason",FORMAT,##__VA_ARGS__);

	extern "C"
	JNIEXPORT void JNICALL
	Java_com_mytest_ffmpegdemo_MainActivity_open(
        JNIEnv *env,
        jobject obj, jstring inputStr_, jstring outStr_) {

    //获取mp4文件路径
    const char *inputStr = env->GetStringUTFChars(inputStr_, 0);
    //获取.yuv的输出文件名
    const char *outStr = env->GetStringUTFChars(outStr_, 0);
    
    //注册各大组件
    av_register_all();

    AVFormatContext *pContext = avformat_alloc_context();

    if (avformat_open_input(&pContext, inputStr, NULL, NULL) < 0) {
        LOGE("打开失败");
        return;
    }
    if (avformat_find_stream_info(pContext, NULL) < 0) {
        LOGE("获取信息失败");
        return;
    }

    int vedio_stream_idx = -1;
    //找到视频流
    for (int i = 0; i < pContext->nb_streams; ++i) {
        LOGE("循环  %d", i);
        //codec 每一个流 对应的解码上下文   codec_type 流的类型
        if (pContext->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            vedio_stream_idx = i;
        }
    }

    //获取到解码器上下文
    AVCodecContext *pCodecCtx = pContext->streams[vedio_stream_idx]->codec;

    //解码器
    AVCodec *pCodex = avcodec_find_decoder(pCodecCtx->codec_id);
    //ffempg版本升级
    if (avcodec_open2(pCodecCtx, pCodex, NULL) < 0) {
        LOGE("解码失败");
        return;
    }
    //分配内存   malloc  AVPacket   1   2
    AVPacket *packet = (AVPacket *) av_malloc(sizeof(AVPacket));
    //初始化结构体
    av_init_packet(packet);


    AVFrame *frame = av_frame_alloc();
    //声明一个yuvframe
    AVFrame *yuvFrame = av_frame_alloc();
    //给yuvframe  的缓冲区 初始化
    uint8_t *out_buffer = (uint8_t *) av_malloc(
            avpicture_get_size(AV_PIX_FMT_YUV420P, pCodecCtx->width, pCodecCtx->height));

    int re = avpicture_fill((AVPicture *) yuvFrame, out_buffer, AV_PIX_FMT_YUV420P,
                            pCodecCtx->width, pCodecCtx->height);
    LOGE("宽 %d  高 %d", pCodecCtx->width, pCodecCtx->height);


    //mp4的上下文pCodecCtx->pix_fmt
    SwsContext *swsContext = sws_getContext(pCodecCtx->width, pCodecCtx->height, pCodecCtx->pix_fmt,
                                            pCodecCtx->width, pCodecCtx->height, AV_PIX_FMT_YUV420P,
                                            SWS_BILINEAR, NULL, NULL, NULL
    );
    int frameCount = 0;
    FILE *fp_yuv = fopen(outStr, "wb");

    //packet入参 出参对象  转换上下文
    int got_frame;
    while (av_read_frame(pContext, packet) >= 0) {
        //解封装
        //根据frame 进行原生绘制    bitmap  window
        avcodec_decode_video2(pCodecCtx, frame, &got_frame, packet);

        //frame的数据拿到视频像素数据yuv
        //为什么不采用RGB呢? 1.RGB数据量大  2.RGB需要三个独立的视频信号同时传输,占用频宽
        LOGE("解码%d  ", frameCount++);
        if (got_frame > 0) {
            sws_scale(swsContext, (const uint8_t *const *) frame->data, frame->linesize, 0,
                      frame->height, yuvFrame->data,
                      yuvFrame->linesize
            );
            int y_size = pCodecCtx->width * pCodecCtx->height;
            //y 亮度信息写完了
            fwrite(yuvFrame->data[0], 1, y_size, fp_yuv);
            fwrite(yuvFrame->data[1], 1, y_size / 4, fp_yuv);
            fwrite(yuvFrame->data[2], 1, y_size / 4, fp_yuv);
        }
        av_free_packet(packet);
    }
    fclose(fp_yuv);
    av_frame_free(&frame);
    av_frame_free(&yuvFrame);
    avcodec_close(pCodecCtx);
    avformat_free_context(pContext);
    env->ReleaseStringUTFChars(inputStr_, inputStr);
    env->ReleaseStringUTFChars(outStr_, outStr);
	}
	

流程就是：

* 1.先注册各大组件
* 2.然后拿到解码器和解码器上下文
* 3.av_read_frame，不停的读取av_packet, 然后解封装成frame，然后将frame转换成yuv,最后写入文件



YUV420数据格式

YUV简介

YUV定义：分为三个分量，

* “Y”表示明亮度（Luminance或Luma）也就是灰度值
* 而“U”和“V” 表示的则是色度（Chrominance或Chroma），作用是描述影像色彩及饱和度，用于指定像素的颜色。
* YUV存储：格式其实与其采样的方式密切相关，主流的采样方式有三种，YUV4:4:4，YUV4:2:2，YUV4:2:0，
* YUV特点：也是一种颜色编码方法，它将亮度信息（Y）与色彩信息（UV）分离，没有UV信息一样 可以显示完整的图像，只不过是黑白的，这样的设计很好地解决了彩色电视机与黑白电视的兼容问题。并且，YUV不像RGB那样要求三个独立的视频信号同时传 输，所以用YUV方式传送占用极少的频宽。


	


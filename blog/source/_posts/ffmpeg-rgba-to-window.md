---
title: FFmpeg将视频文件输出到手机屏幕
date: 2017-10-17 15:57:43
categories: "音视频"
tags:
	- "ffmpeg"
---

<font size=4>

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
	#include <android/native_window_jni.h>
	#include <unistd.h>
	}

	#define LOGI(FORMAT,...) __android_log_print(ANDROID_LOG_INFO,"jason",FORMAT,##__VA_ARGS__);
	#define LOGE(FORMAT,...) __android_log_print(ANDROID_LOG_ERROR,"jason",FORMAT,##__VA_ARGS__);
	extern "C"
	JNIEXPORT void JNICALL
	Java_com_mytest_ffmpegdemo_VideoView_render(JNIEnv *env, jobject instance, jstring input_,
                                             jobject surface) {
    const char *input = env->GetStringUTFChars(input_,false);
    av_register_all();

    AVFormatContext *pFormatCtx = avformat_alloc_context();
    //第四个参数是 可以传一个 字典   是一个入参出参对象
    if (avformat_open_input(&pFormatCtx, input, NULL, NULL) != 0) {
        LOGE("%s","打开输入视频文件失败");
    }
    //3.获取视频信息
    if(avformat_find_stream_info(pFormatCtx,NULL) < 0){
        LOGE("%s","获取视频信息失败");
        return;
    }


    int vidio_stream_idx=-1;
    int i=0;
    for (int i = 0; i < pFormatCtx->nb_streams; ++i) {
        if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            LOGE("  找到视频id %d", pFormatCtx->streams[i]->codec->codec_type);
            vidio_stream_idx=i;
            break;
        }
    }

    //获取视频编解码器
    AVCodecContext *pCodecCtx=pFormatCtx->streams[vidio_stream_idx]->codec;
    LOGE("获取视频编码器上下文 %p  ",pCodecCtx);
    //加密的用不了
    AVCodec *pCodex = avcodec_find_decoder(pCodecCtx->codec_id);
    LOGE("获取视频编码 %p",pCodex);
    //版本升级了
    if (avcodec_open2(pCodecCtx, pCodex, NULL)<0) {


    }
    AVPacket *packet = (AVPacket *)av_malloc(sizeof(AVPacket));
    //av_init_packet(packet);
    //像素数据
    AVFrame *frame;
    frame = av_frame_alloc();
    //RGB
    AVFrame *rgb_frame = av_frame_alloc();
    //给缓冲区分配内存
    //只有指定了AVFrame的像素格式、画面大小才能真正分配内存
    //缓冲区分配内存
    uint8_t   *out_buffer= (uint8_t *) av_malloc(avpicture_get_size(AV_PIX_FMT_RGBA, pCodecCtx->width, pCodecCtx->height));
    LOGE("宽  %d,  高  %d  ",pCodecCtx->width,pCodecCtx->height);
    //设置yuvFrame的缓冲区，像素格式
    int re= avpicture_fill((AVPicture *) rgb_frame, out_buffer, AV_PIX_FMT_RGBA, pCodecCtx->width, pCodecCtx->height);
    LOGE("申请内存%d   ",re);

    //输出需要改变
    int length=0;
    int got_frame;
    //输出文件
    int frameCount=0;
    SwsContext *swsContext = sws_getContext(pCodecCtx->width,pCodecCtx->height,pCodecCtx->pix_fmt,
                                            pCodecCtx->width,pCodecCtx->height,AV_PIX_FMT_RGBA,SWS_BICUBIC,NULL,NULL,NULL
    );
    ANativeWindow *nativeWindow = ANativeWindow_fromSurface(env, surface);
    //视频缓冲区
    ANativeWindow_Buffer outBuffer;
    //ANativeWindow
    while (av_read_frame(pFormatCtx, packet)>=0) {
        //AvFrame
        if (packet->stream_index == vidio_stream_idx) {
            length = avcodec_decode_video2(pCodecCtx, frame, &got_frame, packet);
            LOGE(" 获得长度   %d ", length);

            //非零 正在解码
            if (got_frame) {
                //绘制之前   配置一些信息  比如宽高   格式
                ANativeWindow_setBuffersGeometry(nativeWindow, pCodecCtx->width, pCodecCtx->height,
                                                 WINDOW_FORMAT_RGBA_8888);
                //第一步: 加锁  绘制
                ANativeWindow_lock(nativeWindow, &outBuffer, NULL);
                LOGI("解码%d帧",frameCount++);

                //转为指定的RGBA格式
                sws_scale(swsContext, (const uint8_t *const *) frame->data, frame->linesize, 0
                        , pCodecCtx->height, rgb_frame->data,
                          rgb_frame->linesize);

                //第二步: 缓冲区赋值
                //rgb_frame是有画面数据
                uint8_t *dst= (uint8_t *) outBuffer.bits;
                //拿到一行有多少个字节 RGBA
                int destStride=outBuffer.stride*4;
                //像素数据的首地址
                uint8_t * src= (uint8_t *) rgb_frame->data[0];
                //实际内存一行数量
                int srcStride = rgb_frame->linesize[0];
                int i=0;
                for (int i = 0; i < pCodecCtx->height; ++i) {
                    //memcpy(void *dest, const void *src, size_t n)
                    memcpy(dst + i * destStride,  src + i * srcStride, srcStride);
                }

                //第三步: 解锁
                ANativeWindow_unlockAndPost(nativeWindow);
                usleep(1000 * 16);
            }
        }
        av_free_packet(packet);
    }
    ANativeWindow_release(nativeWindow);
    av_frame_free(&frame);
    avcodec_close(pCodecCtx);
    avformat_free_context(pFormatCtx);
    env->ReleaseStringUTFChars(input_, input);
	}
	
这个是ffmpeg解封装视频文件，转化成RGBA格式，然后赋值给ANativeWindow_Buffer缓冲区，通过ANativeWindow来显示(ANativeWindow需要surface对象)

显示的步骤主要分三步：
	
	//第一步: 加锁  绘制
    ANativeWindow_lock(nativeWindow, &outBuffer, NULL);
    
    //第二步: 缓冲区赋值
    
    //第三步: 解锁
    ANativeWindow_unlockAndPost(nativeWindow);	

注意第三部解锁之后需要休眠 usleep(1000 * 16);
如果不休眠的话，一瞬间就都播放完了。    
    

	
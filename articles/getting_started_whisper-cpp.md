---
title: "whisper.cppをm1 macbook proで動かす方法"
emoji: "🎈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["whisper", "whispercpp", "AI"]
published: true
---

## 前書き
- いわゆる「やってみた」記事です
- 別にこの記事を読まなくてもREADMEをちゃんと読めば十分理解できるはずですが，日本語での情報としてまとめ直すことに一定の意味があると思い記事を書いています．

## 前提
- gitがインストールされている
- homebrewがインストールされている
- ffmpegがインストールされている
    - されていない場合，`brew install ffmpeg`でinstallできます


## whisper(.cpp)とは
- whisper
    - OpenAIという団体が開発したモデル
    - https://github.com/openai/whisper
    - > Whisper is a general-purpose speech recognition model. It is trained on a large dataset of diverse audio and is also a multi-task model that can perform multilingual speech recognition as well as speech translation and language identification.
    - つまり，汎用音声
- whisper.cpp
    - whisperをC/C++移植したもの（高速化したもの）

## 手順概要
- repositoryを手元にclone
- 実行準備
    - build
    - modelのダウンロード
    - 音声ファイルサンプルのダウンロード
- 実行
    - iPhoneで録音してみる
    - AirDropで転送
    - ffmpegでwavに変換
    - 解析実行


## 手順詳細
- `git clone https://github.com/ggerganov/whisper.cpp.git`
- 実行準備 (READMEにしたがって，quickstartを体験する)
    - モデルのダウンロード
        - `bash ./models/download-ggml-model.sh base.en`
    - whisper.cpp自体のbuild
        - `make`
    - 音声ファイルサンプルのダウンロード
        - `make samples`
    - サンプルに対して実行してみる
        - `./main -f samples/gb0.wav`
- 実行
    - iPhoneで録音してみる & AirDropで転送
        - ![](/images/getting_started_whisper-cpp/20221212_screenshot_iphone.jpeg =300x)
    - ffmpegでwavに変換
        - インプットの形式がwavかつサンプリングレートが決まっているのでそれに合わせルように変換
        - `ffmpeg -i ~/Downloads/テスト.m4a -ar 16000 samples/test.wav`
    - 解析実行
        - `./main -m models/ggml-large.bin -l ja -f samples/test.wav`
        - ![](/images/getting_started_whisper-cpp/execute-result.png)
    - 結果
        :::details 実行結果
        ```
        ➜  whisper.cpp git:(master) ./main -m models/ggml-large.bin -l ja -f samples/test.wav
        whisper_model_load: loading model from 'models/ggml-large.bin'
        whisper_model_load: n_vocab       = 51865
        whisper_model_load: n_audio_ctx   = 1500
        whisper_model_load: n_audio_state = 1280
        whisper_model_load: n_audio_head  = 20
        whisper_model_load: n_audio_layer = 32
        whisper_model_load: n_text_ctx    = 448
        whisper_model_load: n_text_state  = 1280
        whisper_model_load: n_text_head   = 20
        whisper_model_load: n_text_layer  = 32
        whisper_model_load: n_mels        = 80
        whisper_model_load: f16           = 1
        whisper_model_load: type          = 5
        whisper_model_load: adding 1608 extra tokens
        whisper_model_load: mem_required  = 4712.00 MB
        whisper_model_load: ggml ctx size = 2950.97 MB
        whisper_model_load: memory size   =  304.38 MB
        whisper_model_load: model size    = 2950.66 MB

        system_info: n_threads = 4 / 8 | AVX = 0 | AVX2 = 0 | AVX512 = 0 | NEON = 1 | F16C = 0 | FP16_VA = 1 | WASM_SIMD = 0 | BLAS = 1 |

        main: processing 'samples/test.wav' (144043 samples, 9.0 sec), 4 threads, 1 processors, lang = ja, task = transcribe, timestamps = 1 ...


        [00:00:01.680 --> 00:00:05.200]  こんばんは 今日のご飯はカレーでした
        [00:00:05.200 --> 00:00:09.200]  辛くて美味しくて良かったです


        whisper_print_timings:     load time =  2369.27 ms
        whisper_print_timings:      mel time =    23.61 ms
        whisper_print_timings:   sample time =     3.89 ms
        whisper_print_timings:   encode time =  9478.08 ms / 296.19 ms per layer
        whisper_print_timings:   decode time =  2259.45 ms / 70.61 ms per layer
        whisper_print_timings:    total time = 14135.59 ms
        ```
        :::details
        - やったね

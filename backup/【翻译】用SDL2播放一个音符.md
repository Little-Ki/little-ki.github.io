[原文](https://blog.fredrb.com/2023/08/08/audio-programming-note-sdl/)

# 频率、振幅和采样率
在使用C代码前先看一些基础信息。
在此实例中，可以看到**周期信号**，是一种在每个时长 $`T`$ 重复的信号，您可能以前看过这样的图像：

![Image](https://github.com/user-attachments/assets/cba1e76f-f76b-4e62-9e00-814afc8922aa)
> 音符 **A4** (440hz) 的**正弦**函数，一条起伏的曲线，用于表示声波。

解释：
周期信号的每次重复称为一次**循环**，**频率**（Hz）是信号完成周期的速率（即其振荡的频率），如果每秒有100个循环，则频率为100Hz。
我们感知到的音符是具有特定频率的声音，在上面的示例中，音符 **A4** 是震荡周期为每秒 440 次的声音，每秒震荡 880 次的声音 **A5** 具有相同的音符 **A** 但高 8 度。

![Image](https://github.com/user-attachments/assets/cfc1e1c0-fedf-42aa-b8a9-febfcc38349a)

> 上表显示了每个音符的频率值。将我们的正弦波频率从 440 更改为 523.3 将从 **A4** 更改为 **C5**。

**振幅** 是波的高度，波峰与波谷的差值，是声音的响度，下面也是音符 **A4** 但具有更小的幅度：

![Image](https://github.com/user-attachments/assets/6f6a0185-53a6-48de-a127-4a7fd1e81940)
> 播放此波形会产生与上面相同但更小的声音。

要编码 **A4** 的正弦波，需要在此波形上采样并送至声卡。正弦函数理论上有无穷多个点，在计算机上只能采样离散数据。**采样率** 是每秒采样数量，更多的采样具有更高的保真度。

缩放查看 2 至 3 毫秒之间的图像，显示了采样率为 44100 时的采样情况：
![Image](https://github.com/user-attachments/assets/951d5d2a-1904-4eeb-b5b5-f71e68a66462)

> 使用高采样率以避免折叠频率，当声音频率过高时，低采样率可能导致两个采样之间具有多个周期，此时无法将高频与较高的音高对应区分开。NYQUIST定理指出，需要至少需要 2 倍于最高频率的采样率来控制此问题。使用44100Hz，可以涵盖人耳可以听到的最高频率。

# SDL音频接口

现在，我们了解了什么是频率以及为什么需要采样率，让我们关注如何使用代码播放 **A4** 音符。
有两种方法告诉声卡要播放什么声音，一种是使用采样**队列**，或者注册回调来发送音频数据，在此示例中使用回调，因为它看起来更简单：
```c
void callback(void* userdata, Uint8* stream, int len) {
    for (int i = 0; i < len; i++) {
        stream[i] = /* ADD SAMPLE HERE */;
    }
}
```

> SDL声音回调：
> - `userdata`：向回调提供自定义数据，可用于在每个回调间传递自定义数据
> - `stream`：由SDL管理的缓冲区，储存了将发送至声卡的采样数据
> - `len`：缓冲区的字节长度

```c
const int SAMPLE_RATE = 44100;
const int BUFFER_SIZE = 4096;

int main() {
    if (SDL_Init(SDL_INIT_AUDIO | SDL_INIT_EVENTS) < 0) {
        printf("Failed to initialize SDL: %s\n", SDL_GetError());
        return 1;
    }

    SDL_AudioSpec spec = {
        .format = AUDIO_F32,
        .channels = 1,
        .freq = SAMPLE_RATE,
        .samples = BUFFER_SIZE,
        1 .callback = callback,
    };

    if (SDL_OpenAudio(&spec, NULL) < 0) {
        printf("Failed to open Audio Device: %s\n", SDL_GetError());
        return 1;
    }

    SDL_PauseAudio(0);
    while (true) {
        SDL_Event e;
        while (SDL_PollEvent(&e)) {
            switch (e.type) {
                case SDL_QUIT:
                    return 0;
            }
        }
    }
    return 0;
}
```

> SDL示例程序将播放由回调提供的音频，直到收到SDL_QUIT

`SDL_AudioSpec.samples` 指定了缓冲区应具有多少帧，也就是回调应提供多少样本，此值将乘以通道数量。
`SDL_AudioSpec.format` 指定了缓冲区的原始类型，默认为8位整数，在此示例中使用了32位浮点数，此时缓冲区具有`4096 * sizeof(float) * spec.channels` 字节数量。
与默认的8位整数相比，将32位浮点数作为采样值提供了更高的精度。正如我们将在下一节中看到的那样，这使得使用SIN功能变得更加容易，因为它已经返回了浮点。

# 在 440Hz 上编写正弦波函数
现在的将编写一个函数，该函数生成了给定频率的正确数量的样本。
- 正弦函数在需要一个弧度 $`\pi`$ 从最低点到最高点，所以需要 $`2\pi`$ 来完成一个周期。
- 要产生音符 **A4**，在一秒内，值 $x$ 将被映射到  $`2\pi`$ 440 次。
- 对于 44100 采样率，每 44100 / 440 的采样 $`x`$ 被映射到  $`2\pi`$。

基于以上条件，需要编写一个产生440Hz正弦波函数的函数。
这是一个简单的通用代码，可以包装任何频率的采样生成：

```c
typedef struct {
    float current_step;
    float step_size;
    float volume;
} oscillator;

oscillator oscillate(float rate, float volume) {
    oscillator o = {
        .current_step = 0,
        .volume = volume,
        .step_size = (2 * M_PI) / rate,
    };
    return o;
}

float next(oscillator * os) {
    os->current_step += os->step_size;
    return sinf(os->current_step) * os->volume;
}
```
> 给出 `rate`，`oscillator` 结构将沿着正弦波产生浮点数

这样，就可以实例化 `oscillator` 并沿正弦波生成浮点数：
```c
float A4_freq = (float) SAMPLE_RATE / 440.00 f;
a4 = oscillate(A4_freq, 0.8 f)

next(a4); // returns the next sample point in the A4 sine wave
```

# 为回调提供样本

现在，我们具有正弦波函数，我们需要在回调中使用它。如上所述，我们使用32位浮点数作为格式。这意味着我们需要做一些工作，以确保我们在提供的缓冲区中写入正确的片段。

```c
void oscillator_callback(void * userdata, Uint8 * stream, int len) {
    float * fstream = (float * ) stream;
    for (int i = 0; i < BUFFER_SIZE; i++) {
        float v = next(A4_oscillator);
        fstream[i] = v;
    }
}
```

必须将缓冲区按照正确的类型写入，错误处理或转换缓冲区可能会导致音频输出混乱，甚至会导致崩溃。
由于 `len` 是字节长度，因此使用 BUFFER_SIZE 来写入正确的数量。

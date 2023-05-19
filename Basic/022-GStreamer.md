# GStreamer

GStreamer是一个多媒体框架，它可以允许你轻易地创建、编辑与播放多媒体文件，这是 通过创建带有很多特殊的多媒体元素的管道来完成的。

Gstreamer是一个支持Windows，Linux，Android， iOS的跨平台的多媒体框架，应用程序可以通过管道（Pipeline）的方式，将多媒体处理的各个步骤串联起来，达到预期的效果。每个步骤通过元素（Element）基于GObject对象系统通过插件（plugins）的方式实现，方便了各项功能的扩展。

## GStreamer 框架

GStreamer 可简单分为multimedia applications,core framework,plugins三层

### multimedia applications

最上面一层为应用，比如gstreamer自带的一些工具（gst-launch，gst-inspect等），以及基于gstreamer封装的库（gst-player，gst-rtsp-server，gst-editing-services等)根据不同场景实现的应用。

### core framework

中间一层为Core Framework，主要提供：

- 上层应用所需接口
- Plugin的框架
- Pipline的框架
- 数据在各个Element间的传输及处理机制
- 多个媒体流（Streaming）间的同步（比如音视频同步）
- 其他各种所需的工具库

### plugins

最下层为各种插件，实现具体的数据处理及音视频输出，应用不需要关注插件的细节，会由Core Framework层负责插件的加载及管理。主要分类为：

- Protocols：负责各种协议的处理，file，http，rtsp等。
- Sources：负责数据源的处理，alsa，v4l2，tcp/udp等。
- Formats：负责媒体容器的处理，avi，mp4，ogg等。
- Codecs：负责媒体的编解码，mp3，vorbis等。
- Filters：负责媒体流的处理，converters，mixers，effects等。
- Sinks：负责媒体流输出到指定设备或目的地，alsa，xvideo，tcp/udp等。

Gstreamer框架根据各个模块的成熟度以及所使用的开源协议，将core及plugins置于不同的源码包中：

- gstreamer: 包含core framework及core elements。
- gst-plugins-base: gstreamer应用所需的必要插件。
- gst-plugins-good: 高质量的采用LGPL授权的插件。
- gst-plugins-ugly: 高质量，但使用了GPL等其他授权方式的库的插件，比如使用GPL的x264，x265。
- gst-plugins-bad: 质量有待提高的插件，成熟后可以移到good插件列表中。
- gst-libav: 对libav封装，使其能在gstreamer框架中使用。

## Gstreamer基础概念

### Element

Element是Gstreamer中最重要的对象类型之一。一个element实现一个功能（读取文件，解码，输出等），程序需要创建多个element，并按顺序将其串连起来，构成一个完整的pipeline。

- **source element** 

  只能生成数据不能接收数据，**例如用于文件读取的filesrc**；

  通常用src pad 表示element 能产生数据，并将其放在element的右边，source element只有src pad,通过设备、文件、网络等方式读取数据后，通过src pad 向 pipeline发送数据，开始pipeline的处理数据。

  ![](C:\Users\hongmei.kuang\Pictures\typora\filesrc.png)

- **sink element**

  只能接受数据不能产生数据的element，例如**用于播放声音的alsasink等**；

  通常用sink pad 表示element 能接收数据，并将其放在element的左边，sink element只有sink  pad ,从sink pad 读取数据后，将数据发送至指定设备或者位置，结束pipeline的处理流程

  ![](C:\Users\hongmei.kuang\Pictures\typora\sink_elment.png)

- **filter-like element**

  既能接收数据又能生成数据的element。例如**分离器、解码器、音量控制等；**

  filter-like element 既包含位于右边的src pad，又包含位于左边的sink pad. element先从左边sink pad 读取数据，然后对数据进行处理，最后在src pad 产生新的数据。

  ![](C:\Users\hongmei.kuang\Pictures\typora\filter_like.png)

  对于这些element，可能包含多个src pad 也可能包含多个sink pad,例如mp4的demuxer(qtdemux)会将mp4文件中的音频和视频分离到audio src pad 和vidio src pad.而mp4 的muxer(mp4mux则相反)，会将audio src pad和vidio src pad的数据合并到一个src pad ，在经过其他element将数据写入文件或者发送到网络。

  ![](C:\Users\hongmei.kuang\Pictures\typora\demuxer.png)

- **连接element**

  当有很多element时，需要将他们按照数据传输路径将其串联起来，src pad 只能连接到sink pad，这样才能实现相对的功能

  ![](C:\Users\hongmei.kuang\Pictures\typora\connect_element.png)

### Pad

pad是一个element的输入/输出接口，分为src pad（生产数据）和sink pad（消费数据）两种。

两个element必须通过pad才能连接起来，pad拥有当前element能处理数据类型的能力（capabilities），会在连接时通过比较src pad和sink pad中所支持的能力，来选择最恰当的数据类型用于传输，如果element不支持，程序会直接退出。在element通过pad连接成功后，数据会从上一个element的src pad传到下一个element的sink pad然后进行处理。

当element支持多种数据处理能力时，可以通过Cap来指定数据类型.

例如，下面的命令通过Cap指定了视频的宽高，videotestsrc会根据指定的宽高产生相应数据：

```
gst-launch-1.0 videotestsrc ! "video/x-raw,width=1280,height=720" ! autovideosink
```

### Bin和Pipeline

Bin是一个容器，用于管理多个element，改变bin的状态时，bin会自动去修改所包含的element的状态，也会转发所收到的消息。如果没有bin，需要依次操作所使用的element。通过bin降低了应用的复杂度。
Pipeline继承自bin，为程序提供一个bus用于传输消息，并且对所有子element进行同步。当将pipeline的状态设置为PLAYING时，pipeline会在一个/多个新的线程中通过element处理数据。

GStreamer 中 element，bin, pipeline 对象之间的继承关系

```
GObject
    ╰──GInitiallyUnowned
        ╰──GstObject
            ╰──GstElement
                ╰──GstBin
                    ╰──GstPipeline
```

这里bin和pipeline都是一个element，那么bin和pipeline都在element的基础上实现了什么功能，解决了什么问题呢？
在创建了element多个element后，需要对element进行状态/资源管理，如果每次状态改变时，都需要依次去操作每个element，这样每次编写一个应用都会有大量的重复工作，这时就有了bin。

Bin继承自element后，实现了容器的功能，可以将多个element添加到bin，当操作bin时，bin会将相应的操作转发到内部所有的element中， 可以将bin认为认为是一个新的逻辑element，由bin来管理其内部element的状态及资源，同时转发其产生的消息。常见的bin有decodebin，autovideoconvert等。

Bin实现了容器的功能，那pipeline又有什么功能呢？

在多媒体应用中，音视频同步是一个基本的功能，需要支持这样的功能，所有的element必须要有一个相同的时钟，这样才能保证各个音频和视频在同一时间输出。pipeline就会为其内部所有的element选择一个相同的时钟，同时还为应用提供了bus系统，用于消息的接收。

下面 通过一个文件播放的例子来熟悉上述提及的概念：测试文件[ sintel_trailer-480p.ogv](http://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.ogv)

```
gst-launch-1.0 filesrc location=sintel_trailer-480p.ogv ! oggdemux name=demux ! queue ! vorbisdec ! autoaudiosink demux. ! queue ! theoradec ! videoconvert ! autovideosink
```

![](C:\Users\hongmei.kuang\Pictures\typora\GStreamer.png)

可以看到这个pipeline由8个element构成，每个element都实现各自的功能：
filesrc读取文件，oggdemux解析文件，分别提取audio，video数据，queue缓存数据，vorbisdec解码audio，autoaudiosink自动选择音频设备并输出，theoradec解码video，videoconvert转换video数据格式，autovideosink自动选择显示设备并输出。

不同的element拥有不同数量及类型的pad，只有src pad的element被称为source element，只有sink pad的被称为sink element。

element可以同时拥有多个相同的pad，例如oggdemux在解析文件后，会将audio，video通过不同的pad输出。

## GStreamer数据消息交互

在pipeline运行的过程中，各个element以及应用之间不可避免的需要进行数据消息的传输，gstreamer提供了bus系统以及多种数据类型（Buffers、Events、Messages，Queries）来达到此目的：

![](C:\Users\hongmei.kuang\Pictures\typora\messages.png)

### bus

Bus是gstreamer内部用于将消息从内部不同的streaming线程，传递到bus线程，再由bus所在线程将消息发送到应用程序。应用程序只需要向bus注册消息处理函数，即可接收到pipline中各element所发出的消息，使用bus后，应用程序就不用关心消息是从哪一个线程发出的，避免了处理多个线程同时发出消息的复杂性。

Bus主要是为了解决多线程之间消息处理的问题。由于GStreamer内部可能会创建多个线程，如果没有bus，应用程序可能同时收到从多个线程的消息，如果应用程序在发送线程中通过回调去处理消息，应用程序有可能阻塞播放线程，造成播放卡顿，死锁等其他问题。

为了解决这类问题，GStreamer通常是将多个线程的消息发送到bus系统，由应用程序从bus中取出消息，然后进行处理。Bus在这里扮演了消息队列的角色，通过bus解耦了GStreamer框架和应用程序对消息的处理，降低了应用程序的复杂度。

### Buffers

用于从sources到sinks的媒体数据传输。

### Events

用于element之间或者应用到element之间的信息传递，比如播放时的seek操作是通过event实现的。

### Messages

是由element发出的消息，通过bus，以异步的方式被应用程序处理。通常用于传递errors, tags, state changes, buffering state, redirects等消息。消息处理是线程安全的。由于大部分消息是通过异步方式处理，所以会在应用程序里存在一点延迟，如果要及时的相应消息，需要在streaming线程捕获处理。

### Queries

用于应用程序向gstreamer查询总时间，当前时间，文件大小等信息。

## GStreaner Tools

Gstreamer自带了gst-inspect-1.0和gst-launch-1.0等其他命令行工具， 可以使用这些工具完成常见的处理任务。

- gst-inspect-1.0
  查看gstreamer的plugin、element的信息。直接将plugin/element的类型作为参数，会列出其详细信息。如果不跟任何参数，会列出当前系统gstreamer所能查找到的所有插件。

  ```
  $ gst-inspect-1.0 playbin
  ```

- gst-launch-1.0
  用于创建及执行一个Pipline，因此通常使用gst-launch先验证相关功能，然后再编写相应应用。
  通过上面ogg视频播放的例子， 已经看到，一个pipeline的多个element之间通过 “!" 分隔，同时可以设置element及Cap的属性。例如：
  播放音视频

```
gst-launch-1.0 playbin file:///home/root/test.mp4
```

转码

```
gst-launch-1.0 filesrc location=/videos/sintel_trailer-480p.ogv ! decodebin name=decode ! \
               videoscale ! "video/x-raw,width=320,height=240" ! x264enc ! queue ! \
               mp4mux name=mux ! filesink location=320x240.mp4 decode. ! audioconvert ! \
               avenc_aac ! queue ! mux.
```

Streaming

```
#Server
gst-launch-1.0 -v videotestsrc ! "video/x-raw,framerate=30/1" ! x264enc key-int-max=30 ! rtph264pay ! udpsink host=127.0.0.1 port=1234

#Client
gst-launch-1.0 udpsrc port=1234 ! "application/x-rtp, payload=96" ! rtph264depay ! decodebin ! autovideosink sync=false
```



# GStreamer----hello world

https://www.cnblogs.com/xleng/p/11008239.html

```
#include <gst/gst.h>

int main (int argc, char *argv[])
{
  GstElement *pipeline;
  GstBus *bus;
  GstMessage *msg;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
  pipeline = gst_parse_launch ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Start playing */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* Free resources */
  if (msg != NULL)
    gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}
```

## 源码分析

### GStreamer初始化

```
  /* Initialize GStreamer */
  gst_init (&argc, &argv);
```

首先调用了gstreamer的初始化函数，该初始化函数必须在其他gstreamer接口之前被调用，gst_init会负责以下资源的初始化：

- 初始化GStreamer库
- 注册内部element
- 加载插件列表，扫描列表中及相应路径下的插件
- 解析并执行命令行参数

在不需要gst_init处理命令行参数时，可以将NULL作为其参数，例如：gst_init(NULL, NULL);

### 创建Pipeline 

```
/* Build the pipeline */
pipeline = gst_parse_launch ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);
```

这一行是示例中的核心逻辑，展示了如何通过gst_parse_launch 创建一个playbin的pipeline，并设置播放文件的uri。

#### gst_parse_launch

在pipeline中，首先通过“source” element获取媒体数据，然后通过一个或多个element对编码数据进行解码，最后通过“sink” element输出声音和画面。

通常在创建较复杂的pipeline时， 需要通过gst_element_factory_make来创建element，然后将其加入到GStreamer Bin中，并连接起来。当pipeline比较简单并且 不需要对pipeline中的element进行过多的控制时， 可以采用gst_parse_launch 来简化pipeline的创建。

这个函数能够巧妙的将pipeline的文本描述转化为pipeline对象， 也经常需要通过文本方式构建pipeline来查看GStreamer是否支持相应的功能，因此GStreamer提供了gst-launch-1.0命令行工具，极大的方便了pipeline的测试。

#### playbin

pipeline中需要添加特定的element以实现相应的功能，在本例中， 通过gst_parse_launch创建了只包含一个element的Pipeline。

 刚提到pipeline需要有“source”、“sink” element，为什么这里只需要一个playbin就够了呢？是因为playbin element内部会根据文件的类型自动去查找所需要的“source”，“decoder”，”sink”并将它们连接起来，同时提供了部分接口用于控制pipeline中相应的element。

在playbin后， 跟了一个uri参数，指定了 想要播放的媒体文件地址，playbin会根据uri所使用的协议（“https://”，“ftp://”，“file://”等）自动选择合适的source element（此例中通过https方式）获取数据。

### 设置播放状态

```
/* Start playing */
gst_element_set_state (pipeline, GST_STATE_PLAYING);
```

这一行代码引入了一个新的概念“状态”（state）。每个GStreamer element都有相应都状态，目前可以简单的把状态与播放器的播放/暂停按钮联系起来，只有当状态处于PLAYING时，pipeline才会播放/处理数据。
这里gst_element_set_state通过pipeline，将playbin的状态设置为PLAYING，使playbin开始播放视频文件。

### 等待播放结束

```
/* Wait until error or EOS */
bus = gst_element_get_bus (pipeline);
msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_ERROR | GST_MESSAGE_EOS);
```

这几行会等待pipeline播放结束或者播放出错。GStreamer框架会通过bus，将所发生的事件通知到应用程序，因此，这里首先取得pipeline的bus对象，通过gst_bus_timed_pop_filtered 以同步的方式等待bus上的ERROR或EOS（End of Stream）消息，该函数收到消息后才会返回。

到目前为止，GStreamer会处理视频播放的所有工作（数据获取，解码，音视频同步，输出）。当到达文件末端（EOS）或出错（直接关闭播放窗口，断开网络）时，播放会自动停止。 也可以在终端通过ctrl+c中断程序的执行。

### 释放资源

```
/* Free resources */
if (msg != NULL)
  gst_message_unref (msg);

gst_object_unref (bus);
gst_element_set_state (pipeline, GST_STATE_NULL);
gst_object_unref (pipeline);
```

将不再使用的msg，bus对象进行销毁，并将pipeline状态设置为NULL（在NULL状态时GStreamer会释放为pipeline分配的所有资源），最后销毁pipeline对象。

由于GStreamer是继承自GObject，所以需要通过gst_object_unref 来减少引用计数，当对象的引用计数为0时，函数内部会自动释放为其分配的内存。

不同接口会对返回的对象进行不同的处理， 需要详细的阅读API文档，来决定 是否需要对返回的对象进行释放。

## element 的创建及使用

```
#include <gst/gst.h>

int main (int argc, char *argv[])
{
  GstElement *pipeline, *source, *filter, *sink;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  source = gst_element_factory_make ("videotestsrc", "source");
  filter = gst_element_factory_make ("timeoverlay", "filter");
  sink = gst_element_factory_make ("autovideosink", "sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !source || !filter || !sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, filter, sink, NULL);
  if (gst_element_link_many (source, filter, sink, NULL) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Modify the source's properties */
  g_object_set (source, "pattern", 0, NULL);

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* Parse message */
  if (msg != NULL) {
    GError *err;
    gchar *debug_info;

    switch (GST_MESSAGE_TYPE (msg)) {
      case GST_MESSAGE_ERROR:
        gst_message_parse_error (msg, &err, &debug_info);
        g_printerr ("Error received from element %s: %s\n",
            GST_OBJECT_NAME (msg->src), err->message);
        g_printerr ("Debugging information: %s\n",
            debug_info ? debug_info : "none");
        g_clear_error (&err);
        g_free (debug_info);
        break;
      case GST_MESSAGE_EOS:
        g_print ("End-Of-Stream reached.\n");
        break;
      default:
        /* We should not reach here because we only asked for ERRORs and EOS */
        g_printerr ("Unexpected message received.\n");
        break;
    }
    gst_message_unref (msg);
  }

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}
```

### 源码分析

创建源码

```
/* Create the elements */
  source = gst_element_factory_make ("videotestsrc", "source");
  filter = gst_element_factory_make ("timeoverlay", "filter");
  sink = gst_element_factory_make ("autovideosink", "sink");
```

在对GStreamer进行初始化后， 可以通过gst_element_factory_make创建element。

第一个参数是element的类型，可以通过这个字符串，找到对应的类型，从而创建element对象。

第二个参数指定了创建element的名字，当 没有保存创建element的对象指针时， 可以通过gst_bin_get_by_name从pipeline中取得该element的对象指针。如果第二个参数为NULL，则GStreamer内部会为该element自动生成一个唯一的名字。
 在当前示例中创建了3个element：videotestsrc，timeoverlay，autovideosink，其作用分别为：

- videotestsrc是一个source element，用于产生视频数据，通常用于调试。
- timeoverlay是一个filter-like element，可以在视频数据中叠加一个时间字符串。
- autovideosink上一个sink element，用于自动选择视频输出设备，创建视频显示窗口，并显示其收到的数据。

### 创建pipeline

```
 /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");
```

Pipeline通过gst_pipeline_new创建，参数为pipeline的名字。

pipeline会提供播放所必须的时钟以及对消息的处理，所以需要把 创建的element添加到pipeline中。

```
/* Build the pipeline */
gst_bin_add_many (GST_BIN (pipeline), source, filter, sink, NULL);
if (gst_element_link_many (source, filter, sink, NULL) != TRUE) {
  g_printerr ("Elements could not be linked.\n");
  gst_object_unref (pipeline);
  return -1;
}
```

pipeline是继承自bin，所以所有bin的方法都可以应用于pipeline，需要注意的是， 需要通过相应的宏（这里是GST_BIN）来将子类转换为父类，宏内部会对其做类型检查。在这里 使用gst_bin_add_many将多个element加入到pipeline中，这个函数接受任意多个参数，最后以NULL表示参数列表的结束。如果一次只需要加入一个，可以使用gst_bin_add函数。

在将element加入bin后， 需要将其连接起来才能完成相应的功能，由于有多个element，所以 这里使用gst_element_link_many，element会根据参数的顺序依次将element连接起来。

**需要注意的是，只有被加入到同一个bin的element才能够被连接在一起，所以 需要在连接前，将所需要的element加入到pipeline/bin中。**

![](C:\Users\hongmei.kuang\Pictures\typora\test_pipelinepng.png)

###  设置element的属性

```
/* Modify the source's properties */
g_object_set (source, "pattern", 0, NULL);
```

大部分的element都有自己的属性。有的属性只能被读取，这种属性常用于查询element的状态。有的属性同时支持修改，这种属性常用于控制element的行为。

由于GstElement继承于GObject，同时GObject对象系统提供了 g_object_get()用于读取属性，g_object_set()用于修改属性，g_object_set()支持以NULL结束的属性-值的键值对，所以可以一次修改element的多个属性。

这里通过g_object_set()来修改videotestsrc的pattern属性。pattern属性可以控制测试图像的类型，可以尝试将0修改为1，查看输出结果有何不同。

可以通过gst-inspect-1.0 videotestsrc命令来查看pattern所支持的所有值。

### 设置播放状态

```
/* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (pipeline);
    return -1;
  }
```

在完成pipeline的创建以及属性的修改后， 将pipeline的状态设置为PLAYING

### 等待播放结束与释放资源

与前面方法一样

# 媒体类型与pad

前面介绍了如何将多个element 连接起来构造一个pipeline，进行数据传输。

但是GStreamer又是通过什么方式来进行数据传输的呢---pad

## pad

pad是element之间的数据的接口，一个src pad只能与一个sink pad相连。每个element可以通过pad过滤数据，接收自己支持的数据类型。Pad通过**Pad Capabilities**（简称为Pad Caps）来描述支持的数据类型。例如：

- 表示分辨率为300x200，帧率为30fps的RGB视频的Caps： 

　　　“video/x-raw,format=RGB,width=300,height=200,framerate=30/1”

- 表示采样位宽为16位，采样率44.1kHz，双通道PCM音频的Caps：

　　　“audio/x-raw,format=S16LE,rate=44100,channels=2”

- 或者直接描述编码数据格式Voribis，VP8：

　　　“audio/x-vorbis” "video/x-vp8"

　　一个Pad可以支持多种类型的Caps（比如一个video sink可以同时支持RGB或YUV格式的数据），同时可以指定Caps支持的数据范围（比如一个audio sink可以支持1~48k的采样率）。但是，在一个Pipeline中，Pad之间所传输的数据类型必须是唯一的。GStreamer在进行element连接时，会通过协商（negotiation）的方式选择一个双方都支持的类型。

　　因此，为了能使两个Element能够正确的连接，双方的Pad Caps之间必须有交集，从而在协商阶段选择相同的数据类型，这就是Pad Caps的主要作用。在实际使用中， 可以通过gst-inspect工具查看Element所支持的Pad Caps，从而才能知道在连接出错时如何处理。

### pad Templates(模板)

 曾使用gst_element_factory_make()接口创建Element，这个接口内部也会先创建一个Element 工厂，再通过工厂方法创建一个Element。由于大部分Element都需要创建类似的Pad，于是GStreame定义了Pad Template，Pad Template被包含中Element工厂中，在创建Element时，用于快速创建Pad。
　　Pad Template包含了一个Pad所能支持的所有Caps。通过Pad Template， 可以快速的判断两个pad是否能够连接（比如两个elements都只提供了sink template，这样的element之间是无法连接的，这样就没必要进一步判断Pad Caps）。

　　由于Pad Template属于Element工厂，所以 可以直接使用gst-inspect查看其属性，但Element实际的Pad会根据Element所处的不同状态来进行实例化，具体的Pad Caps会在协商后才会被确定。

#### Pad Templates Capabilities例子

```
Pad Templates:
  SINK template: 'sink'
    Availability: Always
    Capabilities:
      audio/x-raw
                 format: S16LE
                 layout: interleaved
                   rate: [ 1, 48000 ]
               channels: [ 1, 2 ]
      audio/x-ac3
                 framed: true
```

alsasink只提供了一个sink template，可以创建sink pad，并且是一直存在的。支持两种类型的音频数据：16位的PCM（audio/x-raw），采样率1~48k，1-2通道和AC3（audio/x-ac3）的帧数据。

再看一个 “gst-inspect-1.0 videotestsrc”的例子：

```
Pad Templates:
  SRC template: 'src'
    Availability: Always
    Capabilities:
      video/x-raw
                 format: { I420, YV12, YUY2, UYVY, AYUV, RGBx, BGRx, xRGB, xBGR, RGBA, BGRA, ARGB, ABGR, RGB, BGR, Y41B, Y42B, YVYU, Y444, v210, v216, NV12, NV21, NV16, NV24, GRAY8, GRAY16_BE, GRAY16_LE, v308, RGB16, BGR16, RGB15, BGR15, UYVP, A420, RGB8P, YUV9, YVU9, IYU1, ARGB64, AYUV64, r210, I420_10LE, I420_10BE, I422_10LE, I422_10BE, Y444_10LE, Y444_10BE, GBR, GBR_10LE, GBR_10BE }
                  width: [ 1, 2147483647 ]
                 height: [ 1, 2147483647 ]
              framerate: [ 0/1, 2147483647/1 ]
      video/x-bayer
                 format: { bggr, rggb, grbg, gbrg }
                  width: [ 1, 2147483647 ]
                 height: [ 1, 2147483647 ]
              framerate: [ 0/1, 2147483647/1 ]
```

videotestsrc只提供了一个src template用于创建src pad，pad支持多种格式，可以通过参数指定输出的数据类型或Caps Filter指定。

### Pad Availability

Pad Template都是一直存在的（Availability: Always），创建的Pad也是一直有效的。但有些Element会根据输入数据以及后续的Element动态增加或删除Pad，因此GStreamer提供了3种Pad有效性的状态：Always，Sometimes，On request。

#### Always Pad

在element被初始化后就存在的pad，被称为always pad或static pad。

#### Sometimes Pad

根据输入数据的不同而产生的pad，被称为sometimes pad，常见于各种文件格式解析器。例如用于解析mp4文件的qtdemux："gst-inspect-1.0 qtdemux"

```
Pad Templates:
  SINK template: 'sink'
    Availability: Always
    Capabilities:
      video/quicktime
      video/mj2
      audio/x-m4a
      application/x-3gp

  SRC template: 'video_%u'
    Availability: Sometimes
    Capabilities:
      ANY

  SRC template: 'audio_%u'
    Availability: Sometimes
    Capabilities:
      ANY

  SRC template: 'subtitle_%u'
    Availability: Sometimes
    Capabilities:
      ANY
```

有 从mp4文件中读取数据时， 才能知道这个文件中包含多少音频，视频，字幕，所以这些src pad都是sometimes pad。

#### Request Pad

按需创建的pad被称为request pad，常见于合并或生成多路数据。例如,用于1到N转换的tee："gst-inspect-1.0 tee"

```
Pad Templates:
  ...
  SRC template: 'src_%u'
    Availability: On request
      Has request_new_pad() function: gst_tee_request_new_pad
    Capabilities:
      ANY
```

当 需要将同一路视频流同时进行显示和存储，这时候 就需要用到tee，在创建tee element的时候， 不知道pipeline需要多少个src pad，需要后续element来请求一个src pad。

比如：

GStreamer提供了gst-inspect工具来查看element所提供的Pad Templates，但无法查看element在不同状态时其Pad所支持的数据类型，通过下面的代码， 可以看到Pad Caps在不同状态下的变化。

```
#include <gst/gst.h>

/* Functions below print the Capabilities in a human-friendly format */
static gboolean print_field (GQuark field, const GValue * value, gpointer pfx) {
  gchar *str = gst_value_serialize (value);

  g_print ("%s  %15s: %s\n", (gchar *) pfx, g_quark_to_string (field), str);
  g_free (str);
  return TRUE;
}

static void print_caps (const GstCaps * caps, const gchar * pfx) {
  guint i;

  g_return_if_fail (caps != NULL);

  if (gst_caps_is_any (caps)) {
    g_print ("%sANY\n", pfx);
    return;
  }
  if (gst_caps_is_empty (caps)) {
    g_print ("%sEMPTY\n", pfx);
    return;
  }

  for (i = 0; i < gst_caps_get_size (caps); i++) {
    GstStructure *structure = gst_caps_get_structure (caps, i);

    g_print ("%s%s\n", pfx, gst_structure_get_name (structure));
    gst_structure_foreach (structure, print_field, (gpointer) pfx);
  }
}

/* Prints information about a Pad Template, including its Capabilities */
static void print_pad_templates_information (GstElementFactory * factory) {
  const GList *pads;
  GstStaticPadTemplate *padtemplate;

  g_print ("Pad Templates for %s:\n", gst_element_factory_get_longname (factory));
  if (!gst_element_factory_get_num_pad_templates (factory)) {
    g_print ("  none\n");
    return;
  }

  pads = gst_element_factory_get_static_pad_templates (factory);
  while (pads) {
    padtemplate = pads->data;
    pads = g_list_next (pads);

    if (padtemplate->direction == GST_PAD_SRC)
      g_print ("  SRC template: '%s'\n", padtemplate->name_template);
    else if (padtemplate->direction == GST_PAD_SINK)
      g_print ("  SINK template: '%s'\n", padtemplate->name_template);
    else
      g_print ("  UNKNOWN!!! template: '%s'\n", padtemplate->name_template);

    if (padtemplate->presence == GST_PAD_ALWAYS)
      g_print ("    Availability: Always\n");
    else if (padtemplate->presence == GST_PAD_SOMETIMES)
      g_print ("    Availability: Sometimes\n");
    else if (padtemplate->presence == GST_PAD_REQUEST)
      g_print ("    Availability: On request\n");
    else
      g_print ("    Availability: UNKNOWN!!!\n");

    if (padtemplate->static_caps.string) {
      GstCaps *caps;
      g_print ("    Capabilities:\n");
      caps = gst_static_caps_get (&padtemplate->static_caps);
      print_caps (caps, "      ");
      gst_caps_unref (caps);

    }

    g_print ("\n");
  }
}

/* Shows the CURRENT capabilities of the requested pad in the given element
输出可读信息*/
static void print_pad_capabilities (GstElement *element, gchar *pad_name) {
  GstPad *pad = NULL;
  GstCaps *caps = NULL;

  /* Retrieve pad */
  pad = gst_element_get_static_pad (element, pad_name);
  if (!pad) {
    g_printerr ("Could not retrieve pad '%s'\n", pad_name);
    return;
  }

  /* Retrieve negotiated caps (or acceptable caps if negotiation is not finished yet) */
  caps = gst_pad_get_current_caps (pad);
  if (!caps)
    caps = gst_pad_query_caps (pad, NULL);

  /* Print and free */
  g_print ("Caps for the %s pad:\n", pad_name);
  print_caps (caps, "      ");
  gst_caps_unref (caps);
  gst_object_unref (pad);
}

int main(int argc, char *argv[]) {
  GstElement *pipeline, *source, *sink;
  GstElementFactory *source_factory, *sink_factory;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  gboolean terminate = FALSE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the element factories获取element工厂 */
  source_factory = gst_element_factory_find ("audiotestsrc");
  sink_factory = gst_element_factory_find ("autoaudiosink");
  if (!source_factory || !sink_factory) {
    g_printerr ("Not all element factories could be created.\n");
    return -1;
  }

  /* Print information about the pad templates of these factories */
  print_pad_templates_information (source_factory);
  print_pad_templates_information (sink_factory);

  /* Ask the factories to instantiate actual elements */
  source = gst_element_factory_create (source_factory, "source");
  sink = gst_element_factory_create (sink_factory, "sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !source || !sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);
  if (gst_element_link (source, sink) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Print initial negotiated caps (in NULL state) */
  g_print ("In NULL state:\n");
  print_pad_capabilities (sink, "sink");

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state (check the bus for error messages).\n");
  }

  /* Wait until error, EOS or State Change */
  bus = gst_element_get_bus (pipeline);
  do {
    msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_ERROR | GST_MESSAGE_EOS |
        GST_MESSAGE_STATE_CHANGED);

    /* Parse message */
    if (msg != NULL) {
      GError *err;
      gchar *debug_info;

      switch (GST_MESSAGE_TYPE (msg)) {
        case GST_MESSAGE_ERROR:
          gst_message_parse_error (msg, &err, &debug_info);
          g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
          g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
          g_clear_error (&err);
          g_free (debug_info);
          terminate = TRUE;
          break;
        case GST_MESSAGE_EOS:
          g_print ("End-Of-Stream reached.\n");
          terminate = TRUE;
          break;
        case GST_MESSAGE_STATE_CHANGED://处理state_changed消息
          /* We are only interested in state-changed messages from the pipeline */
          if (GST_MESSAGE_SRC (msg) == GST_OBJECT (pipeline)) {
            GstState old_state, new_state, pending_state;
            gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
            g_print ("\nPipeline state changed from %s to %s:\n",
                gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
            /* Print the current capabilities of the sink element */
            print_pad_capabilities (sink, "sink");
          }
          break;
        default:
          /* We should not reach here because we only asked for ERRORs, EOS and STATE_CHANGED */
          g_printerr ("Unexpected message received.\n");
          break;
      }
      gst_message_unref (msg);
    }
  } while (!terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  gst_object_unref (source_factory);
  gst_object_unref (sink_factory);
  return 0;
}
```

print_field, print_caps and print_pad_templates_information实现类似功能，打印GStreamer的数据结构，可以查看相应GStreamer [GstCaps](https://gstreamer.freedesktop.org/documentation/gstreamer/gstcaps.html?gi-language=c) 接口了解更多信息。

因为 使用的source和sink都具有static（always）pad，所以这里使用gst_element_get_static_pad()获取Pad， 其他情况可以使用gst_element_foreach_pad()或gst_element_iterate_pads()获取动态创建的Pad。
　　接着使用gst_pad_get_current_caps()获取pad当前的caps，根据不同的element状态会有不同的结果，甚至可能不存在caps。如果没有， 通过gst_pad_query_caps()获取当前可以支持的caps，当element处于NULL状态时，这个caps为Pad Template所支持的caps，其值可随状态变化而变化。

#### 输出分析

```
pad Template for Audio test source:
SRC template: 'src'
availability:always
Capabilities:
   audio/x-raw
              format:{ (string)S16LE, (string)S16BE, (string)U16LE, (string)U16BE, (string)S24_32LE, (string)S24_32BE, (string)U24_32LE, (string)U24_32BE, (string)S32LE, (string)S32BE, (string)U32LE, (string)U32BE, (string)S24LE, (string)S24BE, (string)U24LE, (string)U24BE, (string)S20LE, (string)S20BE, (string)U20LE, (string)U20BE, (string)S18LE, (string)S18BE, (string)U18LE, (string)U18BE, (string)F32LE, (string)F32BE, (string)F64LE, (string)F64BE, (string)S8, (string)U8 }
              layout:interleaved
                rate:[ 1, 2147483647 ]
            channels:[ 1, 2147483647 ]

pad Template for Auto audio sink:
SINK template : 'sink' 
availability:always
Capabilities:
   ANY

in null state.
caps for the sink pad:
   ANY

 pipeline state changed from :NULL to :READY
caps for the sink pad:
   audio/x-raw
              format:{ (string)S16LE, (string)S16BE, (string)F32LE, (string)F32BE, (string)S32LE, (string)S32BE, (string)S24LE, (string)S24BE, (string)S24_32LE, (string)S24_32BE, (string)U8 }
              layout:interleaved
                rate:[ 1, 2147483647 ]
            channels:[ 1, 32 ]
   audio/x-alaw
                rate:[ 1, 2147483647 ]
            channels:[ 1, 32 ]
   audio/x-mulaw
                rate:[ 1, 2147483647 ]
            channels:[ 1, 32 ]

 pipeline state changed from :READY to :PAUSED
caps for the sink pad:
   audio/x-raw
              format:S16LE
              layout:interleaved
                rate:44100
            channels:1

 pipeline state changed from :PAUSED to :PLAYING
caps for the sink pad:
   audio/x-raw
              format:S16LE
              layout:interleaved
                rate:44100
            channels:1
```

首先是“audiotestsrc”和“autoaudiosink”的pad templates信息，这个与gst-inspect的输出相同。

NULL状态为Element的初始化状态，此时，“autoaudiosink”的sink pad caps与Pad Template相同，支持所有的格式。

状态从NULL转到READY时，GStreamer会获取音频输出设备所支持的所有类型，这里可以看到sink pad caps列出了输出设备所能支持的类型。

状态从READY转到PAUSED时，GStreamer会协商一个所有element都支持的类型。当进入PLAYING状态时，sink会采用协商后的类型进行数据传输。

# 动态连接pipeline 

根据之前的信息可以知道，有两种播放文件的方式：一是知道了文件的类型及编码方式后，手动创建所需element并构造pipeline，另一种是直接使用playbin,由playbin 内部动态创建所需的element并连接pipeline。 使用palybin 的方式更加灵活，不需要一开始就开始创建各种pipeline，只需有playbin内部根据文件类型，自动构建pipeline。在了解了pad的作用后，接下来通过例子来了解如何通过pad事件动态连接pipeline，为了解playbin内部是如何动态创建pipeline打下基础。

## 动态连接pipeline

在本章的例子中， 在将Pipeline设置为PLAYING状态之前，不会将所有的Element都连接起来，这种处理方式是可以的，但需要额外的处理。

如果在设置PLAYING状态后不做任何操作，数据无法到达Sink，Pipeline会直接抛出一个错误并退出。如果在收到相应事件后，对其进行处理，并将Pipeline连接起来，Pipeline就可以正常工作。

常见的媒体，音频和视频都是通过某一种容器格式被包含中同一个文件中。播放时， 需要将音视频数据分离出来，通常将具备这种功能的模块称为分离器（demuxer）。

GStreamer针对常见的容器提供了相应的demuxer，如果一个容器文件中包含多种媒体数据（例如：一路视频，两路音频），这种情况下，demuxer会为些数据分别创建不同的Source Pad，每一个Source Pad可以被认为一个处理分支，可以创建多个分支分别处理相应的数据。

```
gst-launch-1.0 filesrc location=sintel_trailer-480p.ogv ! oggdemux name=demux ! queue ! vorbisdec ! autoaudiosink demux. ! queue ! theoradec ! videoconvert ! autovideosink
```


通过上面的命令播放文件时，会创建具有2个分支的Pipeline：

![](C:\Users\hongmei.kuang\Pictures\typora\egg1.png)

使用demuxer需要注意的一点是：demuxer只有在收到足够的数据时才能确定容器中包含哪些媒体信息，因此demuxer开始没有Source Pad，所以其他的Element无法在Pipeline创建时就连接到demuxer。

解决这种问题的办法是：在创建Pipeline时， 只将Source Element到demuxer之间的Elements连接好，然后设置Pipeline状态为PLAYING，当demuxer收到足够的数据可以确定文件总包含哪些媒体流时，demuxer会创建相应的Source Pad，并通过事件告诉应用程序。 可以通过监听demuxer的事件，在新的Source Pad被创建时， 根据数据类型，创建相应的Element，再将其连接到Source Pad，形成完整的Pipeline。

## 示例代码

为了简化逻辑，在本示例中会忽略视频的Source Pad，仅连接音频的Source Pad。

```
#include <gst/gst.h>

/* Structure to contain all our information, so we can pass it to callbacks */
typedef struct _CustomData {
  GstElement *pipeline;
  GstElement *source;
  GstElement *convert;
  GstElement *sink;
} CustomData;

/* Handler for the pad-added signal 当Source Element收集到足够到信息，能产生数据时，它会创建Source Pad并且触发“pad-added”信号。这时， 的回调函数就会被调用。*/
static void pad_added_handler (GstElement *src, GstPad *pad, CustomData *data);

int main(int argc, char *argv[]) {
  CustomData data;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  gboolean terminate = FALSE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements 首先创建了所需的Element*/
  data.source = gst_element_factory_make ("uridecodebin", "source");
  data.convert = gst_element_factory_make ("audioconvert", "convert");
  data.sink = gst_element_factory_make ("autoaudiosink", "sink");

  /* Create the empty pipeline */
  data.pipeline = gst_pipeline_new ("test-pipeline");

  if (!data.pipeline || !data.source || !data.convert || !data.sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline. Note that we are NOT linking the source at this
   * point. We will do it later. */
  gst_bin_add_many (GST_BIN (data.pipeline), data.source, data.convert , data.sink, NULL);
  //接着将converter和sink连接起来，注意，这里 没有连接source与convert，是因为uridecode bin在Pipeline初始阶段还没有Source Pad
  if (!gst_element_link (data.convert, data.sink)) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Set the URI to play 这里设置了播放文件的uri，uridecodebin会自动解析该地址，并读取媒体数据。*/
  g_object_set (data.source, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Connect to the pad-added signal 通过g_signal_connect将pad_added_handler回调连接到uridecodebin的“pad-added”信号上，同时附带回调函数的私有参数。*/
  g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);

  /* Start playing */
  ret = gst_element_set_state (data.pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Listen to the bus */
  bus = gst_element_get_bus (data.pipeline);
  do {
    msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
        GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

    /* Parse message */
    if (msg != NULL) {
      GError *err;
      gchar *debug_info;

      switch (GST_MESSAGE_TYPE (msg)) {
        case GST_MESSAGE_ERROR:
          gst_message_parse_error (msg, &err, &debug_info);
          g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
          g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
          g_clear_error (&err);
          g_free (debug_info);
          terminate = TRUE;
          break;
        case GST_MESSAGE_EOS:
          g_print ("End-Of-Stream reached.\n");
          terminate = TRUE;
          break;
        case GST_MESSAGE_STATE_CHANGED:
          /* We are only interested in state-changed messages from the pipeline */
          if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data.pipeline)) {
            GstState old_state, new_state, pending_state;
            gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
            g_print ("Pipeline state changed from %s to %s:\n",
                gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
          }
          break;
        default:
          /* We should not reach here */
          g_printerr ("Unexpected message received.\n");
          break;
      }
      gst_message_unref (msg);
    }
  } while (!terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (data.pipeline, GST_STATE_NULL);
  gst_object_unref (data.pipeline);
  return 0;
}

/* This function will be called by the pad-added signal */
static void pad_added_handler (GstElement *src, GstPad *new_pad, CustomData *data) {
  //首先从CustomData中取得convert指针，并通过gst_element_get_static_pad()获取其Sink Pad。 需要将这个Sink Pad连接到uridecodebin新创建的new_pad中。
  GstPad *sink_pad = gst_element_get_static_pad (data->convert, "sink");
  GstPadLinkReturn ret;
  GstCaps *new_pad_caps = NULL;
  GstStructure *new_pad_struct = NULL;
  const gchar *new_pad_type = NULL;

  g_print ("Received new pad '%s' from '%s':\n", GST_PAD_NAME (new_pad), GST_ELEMENT_NAME (src));

  /* If our converter is already linked, we have nothing to do here */
  由于uridecodebin可能会创建多个Pad，在每次有Pad被创建时， 的回调函数都会被调用。上为了避免重复连接Pad。
  if (gst_pad_is_linked (sink_pad)) {
    g_print ("We are already linked. Ignoring.\n");
    goto exit;
  }

  /* Check the new pad's type 由于 在当前示例中只处理audio相关的数据（ 开始只创建了autoaudiosink），所以 这里对Pad所产生的数据类型进行了过滤，对于非音频Pad（视频及字幕）直接忽略。*/
  new_pad_caps = gst_pad_get_current_caps (new_pad);
  new_pad_struct = gst_caps_get_structure (new_pad_caps, 0);
  new_pad_type = gst_structure_get_name (new_pad_struct);
  if (!g_str_has_prefix (new_pad_type, "audio/x-raw")) {
    g_print ("It has type '%s' which is not raw audio. Ignoring.\n", new_pad_type);
    goto exit;
  }

  /* Attempt the link */
  ret = gst_pad_link (new_pad, sink_pad);
  if (GST_PAD_LINK_FAILED (ret)) {
    g_print ("Type is '%s' but link failed.\n", new_pad_type);
  } else {
    g_print ("Link succeeded (type '%s').\n", new_pad_type);
  }

exit:
  /* Unreference the new pad's caps, if we got them */
  if (new_pad_caps != NULL)
    gst_caps_unref (new_pad_caps);

  /* Unreference the sink pad */
  gst_object_unref (sink_pad);
}
```

## 代码分析

首先创建了所需的Element：

- uridecodebin会中内部实例化所需的Elements（source，demuxer，decoder）将URI所指向的媒体文件中的各种媒体数据分别提取出来。因为其包含了demuxer，所以Source Pad在初始化阶段无法访问，只有在收到相应事件后去动态连接Pad。
- audioconvert用于在不同的音频数据格式之间进行转换。由于不同的声卡支持的数据类型不尽相同，所以在某些平台需要对音频数据类型进行转换。
- autoaudiosink会自动查找声卡设备，并将音频数据传输到声卡上进行输出。

```
/* Create the elements */
data.source = gst_element_factory_make ("uridecodebin", "source");
data.convert = gst_element_factory_make ("audioconvert", "convert");
data.sink = gst_element_factory_make ("autoaudiosink", "sink");
```

接着将converter和sink连接起来，注意，这里 没有连接source与convert，是因为uridecode bin在Pipeline初始阶段还没有Source Pad。

```
if (!gst_element_link (data.convert, data.sink)) {
  g_printerr ("Elements could not be linked.\n");
  gst_object_unref (data.pipeline);
  return -1;
}
```

设置播放文件的uri，uridecodebin会自动解析该地址，并读取媒体数据。

```
/* Set the URI to play */
g_object_set (data.source, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);
```

### 监听事件

GSignals在GStreamer中扮演着至关重要的角色。信号使你能在你所关心到事件发生后得到通知。在GLib中的信号通过信号名来进行识别，每个GObject对象都有其自己的信号。
在上面这行代码中， 通过g_signal_connect将pad_added_handler回调连接到uridecodebin的“pad-added”信号上，同时附带回调函数的私有参数。GStreamer不会处理 传入到data指针，只会将其作为参数传递给回调函数，这是传递私有数据给回调函数的常用方式。
一个GstElement可能会发出多个信号，可以使用gst-inspect工具查看具体到信号及参数。

在 连接了“pad-added”的信号后， 就可以将Pipeline的状态设置为PLAYING并按原有方式处理自己所关心到消息。

```
/* Connect to the pad-added signal */
g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);
```

### 回调处理

当Source Element收集到足够到信息，能产生数据时，它会创建Source Pad并且触发“pad-added”信号。这时， 的回调函数就会被调用。

```
static void pad_added_handler (GstElement *src, GstPad *new_pad, CustomData *data) {
```

这里是 实现到回调函数，为什么 的回调函数需要定义成这种格式呢？
因为 的回调函数是为了处理信号所携带到信息，所以必须用符合信号的数据类型，否则不能正确到处理相应数据。通过gst-inspect查看uridecodebin可以看到信号所需要到回调函数格式：

```
$ gst-inspect-1.0 uridecodebin
...
Element Signals:
  "pad-added" :  void user_function (GstElement* object,
                                     GstPad* arg0,
                                     gpointer user_data);
...
```

- src指针，指向触发这个事件的GstElement对象实例，这里是uridecodebin。GStreamer中的信号处理函数的第一个参数均为触发事件到对象指针。
- new_pad指针，指向被创建的src中被创建的GstPad对象实例。这通常是 需要连接的Pad。
- data指针，指向 在连接信号时所传的CustomData对象

```
GstPad *sink_pad = gst_element_get_static_pad (data->convert, "sink");
```

首先从CustomData中取得convert指针，并通过gst_element_get_static_pad()获取其Sink Pad。 需要将这个Sink Pad连接到uridecodebin新创建的new_pad中。

```
/* If our converter is already linked, we have nothing to do here */
if (gst_pad_is_linked (sink_pad)) {
  g_print ("We are already linked. Ignoring.\n");
  goto exit;
}
```

由于uridecodebin可能会创建多个Pad，在每次有Pad被创建时， 的回调函数都会被调用。上面这段代码就是为了避免重复连接Pad。

```
/* Check the new pad's type */
new_pad_caps = gst_pad_get_current_caps (new_pad, NULL);
new_pad_struct = gst_caps_get_structure (new_pad_caps, 0);
new_pad_type = gst_structure_get_name (new_pad_struct);
if (!g_str_has_prefix (new_pad_type, "audio/x-raw")) {
  g_print ("It has type '%s' which is not raw audio. Ignoring.\n", new_pad_type);
  goto exit;
}
```

由于 在当前示例中只处理audio相关的数据（ 开始只创建了autoaudiosink），所以 这里对Pad所产生的数据类型进行了过滤，对于非音频Pad（视频及字幕）直接忽略。

gst_pad_get_current_caps()可以获取当前Pad的能力（这里是new_pad输出数据的能力），所有的能力被存储在GstCaps结构体中。Pad所支持的所有Caps可以通过gst_pad_query_caps()得到，由于一个Pad可能包含多个Caps，因此GstCaps可以包含一个或多个GstStructure，每个都代表所支持的不同数据的能力。通过gst_pad_get_current_caps()获取到的当前Caps只会包含一个GstStructure用于表示唯一的数据类型，如果无法获取到当前所使用到Caps，该函数会直接返回NULL。

由于 已知在本例中new_pad只包含一个音频Cap，所以 直接通过gst_caps_get_structure()来取得第一个GstStructure。接着再通过gst_structure_get_name() 获取该Cap支持的数据类型，如果不是音频（audio/x-raw）， 直接忽略。

```
/* Attempt the link */
ret = gst_pad_link (new_pad, sink_pad);
if (GST_PAD_LINK_FAILED (ret)) {
  g_print ("Type is '%s' but link failed.\n", new_pad_type);
} else {
  g_print ("Link succeeded (type '%s').\n", new_pad_type);
}
```

对于音频的Source Pad， 使用gst_pad_link()将其与Sink Pad进行连接，使用方式与gst_element_link()相同，指定Source和Sink Pad，其所属的Element必须位于同一个Bin或Pipeline。

到目前为止， 完成了Pipeline的建立，数据会继续在后续的Element中进行音频的播放，直到产生ERROR或EOS。

## GStreamer 的状态

Pipeline在将状态设置为PLAYING之前是不会进入播放状态，实际上PLAYING状态只是GStreamer状态中的一个，GStreamer总共包含4个状态：

1. NULL：NULL状态是所有Element被创建后的初始状态。
2. READY：READY状态表明GStreamer已经完成所需资源的检查，可以进入PAUSED状态。
3. PAUSED：Element处于暂停状态，表明其可以开始接收数据。Sink Element在接收了一个buffer后就会进入等待状态。
4. PLAYING：Element处于播放状态，时钟处于运行中，数据被依次处理。

GStreamer的状态必须按上面的顺序进行切换，例如：不能直接从NULL切换到PLAYING状态，NULL必须依次切换到READY，PAUSED后才能切换到PLAYING状态，当 直接设置Pipeline的状态为PLAYING时，GStreamer内部会依次为 切换到PLAYING状态。

```
case GST_MESSAGE_STATE_CHANGED:
  /* We are only interested in state-changed messages from the pipeline */
  if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data.pipeline)) {
    GstState old_state, new_state, pending_state;
    gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
    g_print ("Pipeline state changed from %s to %s:\n",
        gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
  }
  break;
```

# GStreamer 查询机制

Gstreamer提供了GstQuery的查询机制，用于查询element或pad 的相应信息。例如：查询当前的播放速率，产生的延迟，是否支持跳转等。

要查询所需的信息，首先需要构造与一个查询的类型，然后使用element或pad 的查询接口获取数据，最终在解析相应的结果。比如以下是如何使用GstQuery查询pipeline的总时间

```
GstQuery *query = gst_query_new_duration (GST_FORMAT_TIME);
   gboolean res = gst_element_query (pipeline, query);
   if (res) {
     gint64 duration;
     gst_query_parse_duration (query, NULL, &duration);
     g_print ("duration = %"GST_TIME_FORMAT, GST_TIME_ARGS (duration));
   } else {
     g_print ("duration query failed...");
   }
   gst_query_unref (query);
```

## 示例代码

在本示例中， 通过查询Pipeline是否支持跳转（seeking），如果支持跳转（有些媒体不支持跳转，例如实时视频）， 会在播放10秒后跳转到其他位置。
在以前的示例中， 在Pipeline开始执行后，只等待ERROR和EOS消息，然后退出。本例中， 会在消息等在中设置等待超时时间，超时后， 会去查询当前播放的时间，用于显示，这与播放器的进度条类似。

```
#include <gst/gst.h>

/* Structure to contain all our information, so we can pass it around */
typedef struct _CustomData {
  GstElement *playbin;  /* Our one and only element */
  gboolean playing;      /* Are we in the PLAYING state? */
  gboolean terminate;    /* Should we terminate execution? */
  gboolean seek_enabled; /* Is seeking enabled for this media? */
  gboolean seek_done;    /* Have we performed the seek already? */
  gint64 duration;       /* How long does this media last, in nanoseconds */
} CustomData;

/* Forward definition of the message processing function */
static void handle_message (CustomData *data, GstMessage *msg);

int main(int argc, char *argv[]) {
  CustomData data;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;

  data.playing = FALSE;
  data.terminate = FALSE;
  data.seek_enabled = FALSE;
  data.seek_done = FALSE;
  data.duration = GST_CLOCK_TIME_NONE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.playbin = gst_element_factory_make ("playbin", "playbin");

  if (!data.playbin) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Set the URI to play */
  g_object_set (data.playbin, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Start playing */
  ret = gst_element_set_state (data.playbin, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.playbin);
    return -1;
  }

  /* Listen to the bus */
  bus = gst_element_get_bus (data.playbin);
  do {
    msg = gst_bus_timed_pop_filtered (bus, 100 * GST_MSECOND,
        GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS | GST_MESSAGE_DURATION_CHANGED);

    /* Parse message */
    if (msg != NULL) {
      handle_message (&data, msg);
    } else {
      /* We got no message, this means the timeout expired */
      if (data.playing) {
        gint64 current = -1;

        /* Query the current position of the stream */
        if (!gst_element_query_position (data.playbin, GST_FORMAT_TIME, &current)) {
          g_printerr ("Could not query current position.\n");
        }

        /* If we didn't know it yet, query the stream duration */
        if (!GST_CLOCK_TIME_IS_VALID (data.duration)) {
          if (!gst_element_query_duration (data.playbin, GST_FORMAT_TIME, &data.duration)) {
            g_printerr ("Could not query current duration.\n");
          }
        }

        /* Print current position and total duration */
        g_print ("Position %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
            GST_TIME_ARGS (current), GST_TIME_ARGS (data.duration));

        /* If seeking is enabled, we have not done it yet, and the time is right, seek */
        if (data.seek_enabled && !data.seek_done && current > 10 * GST_SECOND) {
          g_print ("\nReached 10s, performing seek...\n");
          gst_element_seek_simple (data.playbin, GST_FORMAT_TIME,
              GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT, 30 * GST_SECOND);
          data.seek_done = TRUE;
        }
      }
    }
  } while (!data.terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (data.playbin, GST_STATE_NULL);
  gst_object_unref (data.playbin);
  return 0;
}

static void handle_message (CustomData *data, GstMessage *msg) {
  GError *err;
  gchar *debug_info;

  switch (GST_MESSAGE_TYPE (msg)) {
    case GST_MESSAGE_ERROR:
      gst_message_parse_error (msg, &err, &debug_info);
      g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
      g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
      g_clear_error (&err);
      g_free (debug_info);
      data->terminate = TRUE;
      break;
    case GST_MESSAGE_EOS:
      g_print ("End-Of-Stream reached.\n");
      data->terminate = TRUE;
      break;
    case GST_MESSAGE_DURATION_CHANGED:
      /* The duration has changed, mark the current one as invalid */
      data->duration = GST_CLOCK_TIME_NONE;
      break;
    case GST_MESSAGE_STATE_CHANGED: {
      GstState old_state, new_state, pending_state;
      gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
      if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data->playbin)) {
        g_print ("Pipeline state changed from %s to %s:\n",
            gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));

        /* Remember whether we are in the PLAYING state or not */
        data->playing = (new_state == GST_STATE_PLAYING);

        if (data->playing) {
          /* We just moved to PLAYING. Check if seeking is possible */
          GstQuery *query;
          gint64 start, end;
          query = gst_query_new_seeking (GST_FORMAT_TIME);
          if (gst_element_query (data->playbin, query)) {
            gst_query_parse_seeking (query, NULL, &data->seek_enabled, &start, &end);
            if (data->seek_enabled) {
              g_print ("Seeking is ENABLED from %" GST_TIME_FORMAT " to %" GST_TIME_FORMAT "\n",
                  GST_TIME_ARGS (start), GST_TIME_ARGS (end));
            } else {
              g_print ("Seeking is DISABLED for this stream.\n");
            }
          }
          else {
            g_printerr ("Seeking query failed.");
          }
          gst_query_unref (query);
        }
      }
    } break;
    default:
      /* We should not reach here */
      g_printerr ("Unexpected message received.\n");
      break;
  }
  gst_message_unref (msg);
}
```

编译命令

```
gcc basic-tutorial-5.c -o basic-tutorial-5 `pkg-config --cflags --libs gstreamer-1.0`
```

## 代码分析

示例前部分内容与其他示例类似，构造Pipeline并使其进入PLAYING状态。之后开始监听Bus上的消息。

```
msg = gst_bus_timed_pop_filtered (bus, 100 * GST_MSECOND,
    GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS | GST_MESSAGE_DURATION_CHANGED);
```

与以前的示例相比， 在gst_bus_timed_pop_filtered ()中加入了超时时间（100毫秒），这使得此函数如果在100毫秒内没有收到任何消息就会返回超时（msg == NULL）， 会在超时中去更新当前时间，如果返回相应消息（msg != NULL）， 在handle_message中处理相应消息。

```
GStreamer内部有统一的时间类型（GstClockTime），时间计算方式为：GstClockTime = 数值 x 时间单位。GStreamer提供了3种时间单位（宏定义）：GST_SECOND（秒），GST_MSECOND（毫秒），GST_NSECOND（纳秒）。例如：
10秒： 10 * GST_SECOND
100毫秒：100 * GST_MSECOND
100纳秒：100 * GST_NSECOND
```

### 刷新播放时间

首先判断Pipeline的状态，仅在PLAYING状态时才更新当前时间，在非PLAYING状态时查询可能失败。这部分逻辑每秒大概会执行10次，频率足够用于界面的刷新。这里 只将查询到的时间输出到终端。
GstElement封装了相应的接口分别用于查询当前时间（gst_element_query_position）和总时间（gst_element_query_duration ）。

```
/* We got no message, this means the timeout expired */
if (data.playing) {
/* Query the current position of the stream */
if (!gst_element_query_position (data.pipeline, GST_FORMAT_TIME, &current)) {
  g_printerr ("Could not query current position.\n");
}
/* If we didn't know it yet, query the stream duration */
if (!GST_CLOCK_TIME_IS_VALID (data.duration)) {
  if (!gst_element_query_duration (data.pipeline, GST_FORMAT_TIME, &data.duration)) {
     g_printerr ("Could not query current duration.\n");
  }
}
```

使用GST_TIME_FORMAT 和GST_TIME_ARGS 帮助 方便地将GstClockTime的值转换为： ”时：分：秒“格式的字符串输出。

```
/* Print current position and total duration */
g_print ("Position %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
    GST_TIME_ARGS (current), GST_TIME_ARGS (data.duration));
```

同时在超时处理中判断是否需要进行seek操作（在播放到10s时，自动跳转到30s），这里 直接在Pipeline对象上使用gst_element_seek_simple()来执行跳转操作。
gst_element_seek_simple所需要的参数为：

- element ： 需要执行seek操作的Element，这里是Pipeline。
- format：执行seek的类型，这里使用GST_FORMAT_TIME表示 基于时间的方式进行跳转。其他支持的类型可以查看 [GstFormat](https://gstreamer.freedesktop.org/documentation/gstreamer/gstformat.html#GstFormat)。
- seek_flags ：通过标识指定seek的行为。 常用的标识如下，其他支持的flag详见[GstSeekFlags](https://gstreamer.freedesktop.org/documentation/gstreamer/gstsegment.html?gi-language=c#GstSeekFlags)。 

- - GST_SEEK_FLAG_FLUSH：在执行seek前，清除Pipeline中所有buffer中缓存的数据。这可能导致Pipeline在填充的新数据被显示之前出现短暂的等待，但能提高应用更快的响应速度。如果不指定这个标志，Pipeline中的所有缓存数据会依次输出，然后才会播放跳转的位置，会导致一定的延迟。
  - GST_SEEK_FLAG_KEY_UNIT：对于大多数的视频，如果跳转的位置不是关键帧，需要依次解码该帧所依赖的帧（I帧及P帧）后，才能解码此非关键帧。使用这个标识后，seek会自动从最近的I帧开始播放。这个标识降低了seek的精度，提高了seek的效率。
  - GST_SEEK_FLAG_ACCURATE：一些媒体文件没有提供足够的索引信息，在这种文件中执行seek操作会非常耗时，针对这类文件，GStreamer通过内部计算得到需要跳转的位置，大部分的计算结果都是正确的。如果seek的位置不能达到所需精度时，可以增加此标识。但需要注意的是，使用此标识可能会导致seek耗费更多时间来寻找精确的位置。

- seek_pos ：需要跳转的位置，前面指定了seek的类型为时间，所以这里是30秒。

```
/* If seeking is enabled, we have not done it yet, and the time is right, seek */
if (data.seek_enabled && !data.seek_done && current > 10 * GST_SECOND) {
  g_print ("\nReached 10s, performing seek...\n");
  gst_element_seek_simple (data.pipeline, GST_FORMAT_TIME,
      GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT, 30 * GST_SECOND);
  data.seek_done = TRUE;
}
```

### 消息处理

在handle_message接口中处理所有Pipeline上的消息，ERROR和EOS与以前示例处理方式相同，此例中新增了以下内容：

```
case GST_MESSAGE_DURATION_CHANGED:
  /* The duration has changed, mark the current one as invalid */
  data->duration = GST_CLOCK_TIME_NONE;
  break;
```

在文件的总时间发生变化时， 会收到此消息，这里简单的将总长度标记为非法值，在下次更新时间时进行查询。

```
case GST_MESSAGE_STATE_CHANGED: {
  GstState old_state, new_state, pending_state;
  gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
  if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data->pipeline)) {
    g_print ("Pipeline state changed from %s to %s:\n",
        gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));

    /* Remember whether we are in the PLAYING state or not */
    data->playing = (new_state == GST_STATE_PLAYING);
```

跳转和时间查询操作仅在PUASED和PLAYING状态时才能得到正确的结果，因为所有的Element只能在这2个状态才能接收处理seek和query的指令。这里会保存播放的状态便于后续使用，并且在进入PLAYING状态时查询当前所播放的文件/流是否支持跳转操作：

```
if (data->playing) {
  /* We just moved to PLAYING. Check if seeking is possible */
  GstQuery *query;
  gint64 start, end;
  query = gst_query_new_seeking (GST_FORMAT_TIME);
  if (gst_element_query (data->pipeline, query)) {
    gst_query_parse_seeking (query, NULL, &data->seek_enabled, &start, &end);
    if (data->seek_enabled) {
      g_print ("Seeking is ENABLED from %" GST_TIME_FORMAT " to %" GST_TIME_FORMAT "\n",
          GST_TIME_ARGS (start), GST_TIME_ARGS (end));
    } else {
      g_print ("Seeking is DISABLED for this stream.\n");
    }
  }
  else {
    g_printerr ("Seeking query failed.");
  }
  gst_query_unref (query);
}
```

这里的查询步骤与文章开始介绍的方式相同：

- 首先，通过gst_query_new_seeking()构造一个跳转的查询对象，使用GST_FORMAT_TIME作为参数，表明 需要知道当前的文件是否支持通过时间进行跳转。 同样可以使用GST_FORMAT_BYTES作为参数，用于查询是否可以根据文件的偏移量来就行跳转，但这种使用方式不常见。
- 接着，将查询对象传入gst_element_query()查询，并得到结果。
- 最后，通过gst_query_parse_seeking()解析是否支持跳转及所支持的范围。

**在使用完后释放查询对象**。

# 获取媒体信息

在常见的媒体文件中，通常包含一些数据（例如：歌手，专辑，编码类型等），用于描述媒体文件。

通常称这些数据为元数据（Metadata：data that provides information about other data）。

可以通过这些元数据对媒体进行归类，同时可以在播放的过程中通过界面显示。

接下里将介绍GStreamer是如何快速获取元数据。

## GStreamer 元数据

GStreamer将元数据分为了两类：

- 流信息（Stream-info）：用于描述流的属性。例如：编码类型、分辨率、采样率等。

  Stream-info可以通过pipeline中所有的GstCaps获取，使用方式再媒体类型与pad中有描述

- 流标签（stream-tag）：用于描述非技术性的信息，例如：作者、标题、专辑等。

  Stream-tag可以通过GstBus ,监听GST_MESSAGE_TSG消息，从消息中提取相应的信息。

  需要注意的是，GStreamer可能触发多次GST_MESSAGE_TAG消息，应用程序可以通过gst_tag_list_merge()合并多个标签，再在适当的时间显示，当切换媒体文件时，需要清空缓存。

  使用此函数时，需要采用GST_TAG_MERGE_PERPEND,这样后面更新的元数据会有更高的优先级。

## 示例代码

```
#include <gst/gst.h>

static void
print_one_tag (const GstTagList * list, const gchar * tag, gpointer user_data)
{
  int i, num;

  num = gst_tag_list_get_tag_size (list, tag);
  for (i = 0; i < num; ++i) {
    const GValue *val;

    /* Note: when looking for specific tags, use the gst_tag_list_get_xyz() API,
     * we only use the GValue approach here because it is more generic */
    val = gst_tag_list_get_value_index (list, tag, i);
    if (G_VALUE_HOLDS_STRING (val)) {
      g_print ("\t%20s : %s\n", tag, g_value_get_string (val));
    } else if (G_VALUE_HOLDS_UINT (val)) {
      g_print ("\t%20s : %u\n", tag, g_value_get_uint (val));
    } else if (G_VALUE_HOLDS_DOUBLE (val)) {
      g_print ("\t%20s : %g\n", tag, g_value_get_double (val));
    } else if (G_VALUE_HOLDS_BOOLEAN (val)) {
      g_print ("\t%20s : %s\n", tag,
          (g_value_get_boolean (val)) ? "true" : "false");
    } else if (GST_VALUE_HOLDS_BUFFER (val)) {
      GstBuffer *buf = gst_value_get_buffer (val);
      guint buffer_size = gst_buffer_get_size (buf);

      g_print ("\t%20s : buffer of size %u\n", tag, buffer_size);
    } else if (GST_VALUE_HOLDS_DATE_TIME (val)) {
      GstDateTime *dt = g_value_get_boxed (val);
      gchar *dt_str = gst_date_time_to_iso8601_string (dt);

      g_print ("\t%20s : %s\n", tag, dt_str);
      g_free (dt_str);
    } else {
      g_print ("\t%20s : tag of type '%s'\n", tag, G_VALUE_TYPE_NAME (val));
    }
  }
}

static void
on_new_pad (GstElement * dec, GstPad * pad, GstElement * fakesink)
{
  GstPad *sinkpad;

  sinkpad = gst_element_get_static_pad (fakesink, "sink");
  if (!gst_pad_is_linked (sinkpad)) {
    if (gst_pad_link (pad, sinkpad) != GST_PAD_LINK_OK)
      g_error ("Failed to link pads!");
  }
  gst_object_unref (sinkpad);
}

int
main (int argc, char ** argv)
{
  GstElement *pipe, *dec, *sink;
  GstMessage *msg;
  gchar *uri;

  gst_init (&argc, &argv);

  if (argc < 2)
    g_error ("Usage: %s FILE or URI", argv[0]);

  if (gst_uri_is_valid (argv[1])) {
    uri = g_strdup (argv[1]);
  } else {
    uri = gst_filename_to_uri (argv[1], NULL);
  }

  pipe = gst_pipeline_new ("pipeline");

  dec = gst_element_factory_make ("uridecodebin", NULL);
  g_object_set (dec, "uri", uri, NULL);
  gst_bin_add (GST_BIN (pipe), dec);

  sink = gst_element_factory_make ("fakesink", NULL);
  gst_bin_add (GST_BIN (pipe), sink);

  g_signal_connect (dec, "pad-added", G_CALLBACK (on_new_pad), sink);

  gst_element_set_state (pipe, GST_STATE_PAUSED);

  while (TRUE) {
    GstTagList *tags = NULL;

    msg = gst_bus_timed_pop_filtered (GST_ELEMENT_BUS (pipe),
        GST_CLOCK_TIME_NONE,
        GST_MESSAGE_ASYNC_DONE | GST_MESSAGE_TAG | GST_MESSAGE_ERROR);

    if (GST_MESSAGE_TYPE (msg) != GST_MESSAGE_TAG) /* error or async_done */
      break;

    gst_message_parse_tag (msg, &tags);

    g_print ("Got tags from element %s:\n", GST_OBJECT_NAME (msg->src));
    gst_tag_list_foreach (tags, print_one_tag, NULL);
    g_print ("\n");
    gst_tag_list_unref (tags);

    gst_message_unref (msg);
  }

  if (GST_MESSAGE_TYPE (msg) == GST_MESSAGE_ERROR) {
    GError *err = NULL;

    gst_message_parse_error (msg, &err, NULL);
    g_printerr ("Got error: %s\n", err->message);
    g_error_free (err);
  }

  gst_message_unref (msg);
  gst_element_set_state (pipe, GST_STATE_NULL);
  gst_object_unref (pipe);
  g_free (uri);
  return 0;
}
```

将源码保存为basic-tutorial-6.c，执行下列命令可得到编译结果：

````
gcc basic-tutorial-6.c -o basic-tutorial-6 `pkg-config --cflags --libs gstreamer-1.0
````

 示例输出

```
$ ./basic-tutorial-6 sintel_trailer-480p.ogv
Got tags from element fakesink0:
                       title : Sintel Trailer
                      artist : Durian Open Movie Team
                   copyright : (c) copyright Blender Foundation | durian.blender.org
                     license : Creative Commons Attribution 3.0 license
            application-name : ffmpeg2theora-0.24
                     encoder : Xiph.Org libtheora 1.1 20090822 (Thusnelda)
                 video-codec : Theora
             encoder-version : 3

Got tags from element fakesink0:
            container-format : Ogg
```

## 代码分析

本例中使用uridecodebin解析媒体文件，Pipeline的构造与其他示例相同，下面介绍Tag相关的处理逻辑。

```
static void
print_one_tag (const GstTagList * list, const gchar * tag, gpointer user_data)
{
  int i, num;

  num = gst_tag_list_get_tag_size (list, tag);
  for (i = 0; i < num; ++i) {
    const GValue *val;

    /* Note: when looking for specific tags, use the gst_tag_list_get_xyz() API,
     * we only use the GValue approach here because it is more generic */
    val = gst_tag_list_get_value_index (list, tag, i);
    if (G_VALUE_HOLDS_STRING (val)) {
      g_print ("\t%20s : %s\n", tag, g_value_get_string (val));
    } 
...
}
```

此函数用于输出一个标签的值。GStreamer会将多个标签都放在同一个GstTagList中。

每一个标签可以包含多个值，所以首先通过gst_tag_list_get_tag_size ()接口及标签名（tag）获取其值的数量，然后再获取相应的值。
　　本例使用GValue来进行通用的处理，所以需要先判断数据的类型，再通过GValue接口获取。实际处理标签时，可以根据规范（例如ID3Tag）得到标签值的类型，直接通过GstTagList接口获取，例如：当标签名为title时， 可以直接使用gst_tag_list_get_string()取得title的字符串，不需要再通过GValue转换，详细使用方式可参考[GstTagList文档](https://gstreamer.freedesktop.org/documentation/gstreamer/gsttaglist.html?gi-language=c)。

```
static void
on_new_pad (GstElement * dec, GstPad * pad, GstElement * fakesink)
{
  GstPad *sinkpad;

  sinkpad = gst_element_get_static_pad (fakesink, "sink");
  if (!gst_pad_is_linked (sinkpad)) {
    if (gst_pad_link (pad, sinkpad) != GST_PAD_LINK_OK)
      g_error ("Failed to link pads!");
  }
  gst_object_unref (sinkpad);
}
...
  sink = gst_element_factory_make ("fakesink", NULL);
  gst_bin_add (GST_BIN (pipe), sink);
  g_signal_connect (dec, "pad-added", G_CALLBACK (on_new_pad), sink);
```

由于只需要提取相应的媒体信息，不需要关心具体的数据，所以这里使用了fakesink，fakesink会直接丢弃掉所有收到的数据。同时在此处监听了"pad-added"的信号，用于动态连接Pipeline，这种处理方式已在[动态连接Pipeline](https://www.cnblogs.com/xleng/p/11194226.html)中进行了详细的介绍。

```
while (TRUE) {
    GstTagList *tags = NULL;

    msg = gst_bus_timed_pop_filtered (GST_ELEMENT_BUS (pipe),
        GST_CLOCK_TIME_NONE,
        GST_MESSAGE_ASYNC_DONE | GST_MESSAGE_TAG | GST_MESSAGE_ERROR);

    if (GST_MESSAGE_TYPE (msg) != GST_MESSAGE_TAG) /* error or async_done */
      break;

    gst_message_parse_tag (msg, &tags);

    g_print ("Got tags from element %s:\n", GST_OBJECT_NAME (msg->src));
    gst_tag_list_foreach (tags, print_one_tag, NULL);
    g_print ("\n");
    gst_tag_list_unref (tags);

    gst_message_unref (msg);
  }
```

与其他示例相同，这里也采用gst_bus_timed_pop_filtered()获取Bus上的GST_MESSAGE_TAG，再通过gst_message_parse_tag ()从消息中将标签拷贝到GstTagList中，再通过gst_tag_list_foreach ()依次输出所有的标签，随后释放GstTagList。

**需要注意的是**，如果GstTagList中不包含任何标签信息，gst_tag_list_foreach ()中的回调函数不会被调用。

Stream-tag主要是通过监听GST_MESSAGE_TAG后，根据相应接口提取元数据。在使用的过程中需要注意数据的释放。

## GstDiscoverer

获取媒体信息是一个常用的功能，因此GStreamer通过[GstDiscoverer](https://gstreamer.freedesktop.org/documentation/pbutils/gstdiscoverer.html?gi-language=c)提供了一组实用接口。使用时无需关心内部Pipeline的创建，只需通过gst_discoverer_new()创建实例，使用gst_discoverer_discover_uri()指定URI，监听相应信号后，即可在回调函数中得到相应的元数据，使用时需要额外连接libgstpbutils-1.0库。

GStreamer同时基于GstDiscoverer提供了gst-discoverer-1.0工具，使用方式如下：

```
$ gst-discoverer-1.0 sintel_trailer-480p.mp4
Analyzing file:///home/xleng/video/sintel_trailer-480p.mp4
Done discovering file:///home/xleng/video/sintel_trailer-480p.mp4

Topology:
  container: Quicktime
    audio: MPEG-4 AAC
    video: H.264 (High Profile)

Properties:
  Duration: 0:00:52.209000000
  Seekable: yes
  Live: no
  Tags:
      audio codec: MPEG-4 AAC audio
      maximum bitrate: 128000
      datetime: 1970-01-01T00:00:00Z
      title: Sintel Trailer
      artist: Durian Open Movie Team
      copyright: (c) copyright Blender Foundation | durian.blender.org
      description: Trailer for the Sintel open movie project
      encoder: Lavf52.62.0
      container format: ISO MP4/M4A
      video codec: H.264 / AVC
      bitrate: 535929
```

# 播放速率控制

在常见的媒体播放器中，通常可以看到快进，快退，慢放等功能，这部分功能被称为“特技模式（Trick Mode）”，这些模式有个共同点：都通过修改播放的速率来达到相应的目的。 

一下将介绍如何通过GStreamer去实现快进，快退，慢放以及单帧播放。

## GStreamer Seek与Step事件

快进（Fast-Forward），快退（Fast-Rewind）和慢放（Slow-Motion）都是通过修改播放的速率来达到相应的目的。在GStreamer中，将1倍速作为正常的播放速率，将大于1倍速的2倍，4倍，8倍等倍速称为快进，慢放则是播放速率的绝对值小于1倍速，当播放速率小于0时，则进行倒放。

在GStreamer中， 通过seek与step事件来控制Element的播放速率及区域。Step事件允许跳过指定的区域并设置后续的播放速率（此速率必须大于0）。Seek事件允许跳转到播放文件中的的任何位置，并且播放速率可以大于0或小于0.
　　在[播放时间控制](https://www.cnblogs.com/xleng/p/11239667.html )中，使用gst_element_seek_simple 来快速的跳转到指定的位置，此函数是对seek事件的封装。实际使用时，首先需要构造一个seek event，设置seek的绝对起始位置和停止位置，停止位置可以设置为0，这样会执行seek的播放速率直到结束。同时可以支持按buffer的方式进行seek，以及设置不同的标志指定seek的行为。
　　Step事件相较于Seek事件需要更少的参数，更易用于修改播放速率，但是不够灵活。Step事件只会作用于最终的sink，Seek事件则可以作用于Pipeline中所有的Element。Step操作的效率高于Seek。
　　在GStreamer中，单帧播放(Frame Stepping）与快进相同，也是通过事件实现。单帧播放通常在暂停的状态下，构造并发送step event每次播放一帧。
　　**需要注意的是**，seek event需要直接作用于sink element（eg： audio sink或video sink），如果直接将seek event作用于Pipeline，Pipeline会自动将事件转发给所有的sink，如果有多个sink，就会造成多次seek。通常是先获取Pipeline中的video-sink或audio-sink，然后发送seek event到指定的sink，完成seek的操作。 Seek时间的构造及发送示例如下：

```
GstEvent *event;
   gboolean result;
   ...
   // construct a seek event to play the media from second 2 to 5, flush
   // the pipeline to decrease latency.
   event = gst_event_new_seek (1.0,
      GST_FORMAT_TIME,
      GST_SEEK_FLAG_FLUSH,
      GST_SEEK_TYPE_SET, 2 * GST_SECOND,
      GST_SEEK_TYPE_SET, 5 * GST_SECOND);
   ...
   result = gst_element_send_event (video_sink, event);
   if (!result)
     g_warning ("seek failed");
   ...
```

## 示例代码

```
#include <string.h>
#include <stdio.h>
#include <gst/gst.h>

typedef struct _CustomData
{
  GstElement *pipeline;
  GstElement *video_sink;
  GMainLoop *loop;

  gboolean playing;             /* Playing or Paused */
  gdouble rate;                 /* Current playback rate (can be negative) */
} CustomData;

/* Send seek event to change rate */
static void send_seek_event (CustomData * data)
{
  gint64 position;
  GstEvent *seek_event;

  /* Obtain the current position, needed for the seek event */
  if (!gst_element_query_position (data->pipeline, GST_FORMAT_TIME, &position)) {
    g_printerr ("Unable to retrieve current position.\n");
    return;
  }

  /* Create the seek event */
  if (data->rate > 0) {
    seek_event =
        gst_event_new_seek (data->rate, GST_FORMAT_TIME,
        GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_ACCURATE, GST_SEEK_TYPE_SET,
        position, GST_SEEK_TYPE_END, 0);
  } else {
    seek_event =
        gst_event_new_seek (data->rate, GST_FORMAT_TIME,
        GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_ACCURATE, GST_SEEK_TYPE_SET, 0,
        GST_SEEK_TYPE_SET, position);
  }

  if (data->video_sink == NULL) {
    /* If we have not done so, obtain the sink through which we will send the seek events */
    g_object_get (data->pipeline, "video-sink", &data->video_sink, NULL);
  }

  /* Send the event */
  gst_element_send_event (data->video_sink, seek_event);

  g_print ("Current rate: %g\n", data->rate);
}

/* Process keyboard input */
static gboolean
handle_keyboard (GIOChannel * source, GIOCondition cond, CustomData * data)
{
  gchar *str = NULL;

  if (g_io_channel_read_line (source, &str, NULL, NULL,
          NULL) != G_IO_STATUS_NORMAL) {
    return TRUE;
  }

  switch (g_ascii_tolower (str[0])) {
    case 'p':
      data->playing = !data->playing;
      gst_element_set_state (data->pipeline,
          data->playing ? GST_STATE_PLAYING : GST_STATE_PAUSED);
      g_print ("Setting state to %s\n", data->playing ? "PLAYING" : "PAUSE");
      break;
    case 's':
      if (g_ascii_isupper (str[0])) {
        data->rate *= 2.0;
      } else {
        data->rate /= 2.0;
      }
      send_seek_event (data);
      break;
    case 'd':
      data->rate *= -1.0;
      send_seek_event (data);
      break;
    case 'n':
      if (data->video_sink == NULL) {
        /* If we have not done so, obtain the sink through which we will send the step events */
        g_object_get (data->pipeline, "video-sink", &data->video_sink, NULL);
      }

      gst_element_send_event (data->video_sink,
          gst_event_new_step (GST_FORMAT_BUFFERS, 1, ABS (data->rate), TRUE,
              FALSE));
      g_print ("Stepping one frame\n");
      break;
    case 'q':
      g_main_loop_quit (data->loop);
      break;
    default:
      break;
  }

  g_free (str);

  return TRUE;
}

int main (int argc, char *argv[])
{
  CustomData data;
  GstStateChangeReturn ret;
  GIOChannel *io_stdin;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Initialize our data structure */
  memset (&data, 0, sizeof (data));

  /* Print usage map */
  g_print ("USAGE: Choose one of the following options, then press enter:\n"
      " 'P' to toggle between PAUSE and PLAY\n"
      " 'S' to increase playback speed, 's' to decrease playback speed\n"
      " 'D' to toggle playback direction\n"
      " 'N' to move to next frame (in the current direction, better in PAUSE)\n"
      " 'Q' to quit\n");

  /* Build the pipeline */
  data.pipeline =
      gst_parse_launch
      ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm",
      NULL);

  /* Add a keyboard watch so we get notified of keystrokes */
#ifdef G_OS_WIN32
  io_stdin = g_io_channel_win32_new_fd (fileno (stdin));
#else
  io_stdin = g_io_channel_unix_new (fileno (stdin));
#endif
  g_io_add_watch (io_stdin, G_IO_IN, (GIOFunc) handle_keyboard, &data);

  /* Start playing */
  ret = gst_element_set_state (data.pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }
  data.playing = TRUE;
  data.rate = 1.0;

  /* Create a GLib Main Loop and set it to run */
  data.loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (data.loop);

  /* Free resources */
  g_main_loop_unref (data.loop);
  g_io_channel_unref (io_stdin);
  gst_element_set_state (data.pipeline, GST_STATE_NULL);
  if (data.video_sink != NULL)
    gst_object_unref (data.video_sink);
  gst_object_unref (data.pipeline);
  return 0;
}
```

通过下面的命令编译即可得到可执行文件，在终端输入相应指令可修改播放速率。

```
gcc basic-tutorial-7.c -o basic-tutorial-7 `pkg-config --cflags --libs gstreamer-1.0`
```

## 代码分析

Pipeline的创建与其他示例相同，通过playbin播放文件，采用GLib的I/O接口来处理键盘输入。

```
/* Process keyboard input */
static gboolean handle_keyboard (GIOChannel *source, GIOCondition cond, CustomData *data) {
  gchar *str = NULL;

  if (g_io_channel_read_line (source, &str, NULL, NULL, NULL) != G_IO_STATUS_NORMAL) {
    return TRUE;
  }

  switch (g_ascii_tolower (str[0])) {
  case 'p':
    data->playing = !data->playing;
    gst_element_set_state (data->pipeline, data->playing ? GST_STATE_PLAYING : GST_STATE_PAUSED);
    g_print ("Setting state to %s\n", data->playing ? "PLAYING" : "PAUSE");
    break;
```

在终端输入P时，使用gst_element_set_state ()设置播放状态。

```
case 's':
  if (g_ascii_isupper (str[0])) {
    data->rate *= 2.0;
  } else {
    data->rate /= 2.0;
  }
  send_seek_event (data);
  break;
case 'd':
  data->rate *= -1.0;
  send_seek_event (data);
  break;
```

　通过S和s增加和降低播放速度，d用于改变播放方向（倒放），这里在修改rate后，调用send_seek_event实现真正的处理。

```
/* Send seek event to change rate */
static void send_seek_event (CustomData *data) {
  gint64 position;
  GstEvent *seek_event;

  /* Obtain the current position, needed for the seek event */
  if (!gst_element_query_position (data->pipeline, GST_FORMAT_TIME, &position)) {
    g_printerr ("Unable to retrieve current position.\n");
    return;
  }
```

这个函数会构造一个SeekEvent发送到Pipeline以调节速率。因为Seek Event会跳转到指定的位置，但 在此例汇总只想改变速率，不跳转到其他位置，所以首先通过gst_element_query_position ()获取当前的播放位置。

```
/* Create the seek event */
if (data->rate > 0) {
  seek_event = gst_event_new_seek (data->rate, GST_FORMAT_TIME, GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_ACCURATE,
      GST_SEEK_TYPE_SET, position, GST_SEEK_TYPE_END, 0);
} else {
  seek_event = gst_event_new_seek (data->rate, GST_FORMAT_TIME, GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_ACCURATE,
      GST_SEEK_TYPE_SET, 0, GST_SEEK_TYPE_SET, position);
}
```

通过gst_event_new_seek()创建SeekEvent，设置新的rate，flag，起始位置，结束位置。需要注意的是，起始位置需要小于结束位置。

```
if (data->video_sink == NULL) {
  /* If we have not done so, obtain the sink through which we will send the seek events */
  g_object_get (data->pipeline, "video-sink", &data->video_sink, NULL);
}
/* Send the event */
gst_element_send_event (data->video_sink, seek_event);
```

为了避免Pipeline执行多次的seek， 在此处获取video-sink，并向其发送SeekEvent。 直到执行Seek时才获取video-sink是因为实际的sink有可能会根据不同的媒体类型，在PLAYING状态时才创建。

　　以上部分内容就是速率的修改，关于单帧播放的情况，实现方式更加简单：

```
case 'n':
  if (data->video_sink == NULL) {
    /* If we have not done so, obtain the sink through which we will send the step events */
    g_object_get (data->pipeline, "video-sink", &data->video_sink, NULL);
  }

  gst_element_send_event (data->video_sink,
      gst_event_new_step (GST_FORMAT_BUFFERS, 1, ABS (data->rate), TRUE, FALSE));
  g_print ("Stepping one frame\n");
  break;
```

通过gst_event_new_step()创建了StepEvent，并指定只跳转一帧，且不改变当前速率。单帧播放通常都是先暂停，然后再进行单帧播放。

　　以上就是通过GStreamer实现播放速率的控制，实际中，有些Element对倒放支持不是很好，不能达到理想的效果。

# GStreamer 多线程

GStreamer框架会自动处理多线程的逻辑，但在某种情况下，仍需要根据实际情况将部分pipeline 再单独的线程中执行。

## GStreamer多线程

GStreamer框架是一个支持多线程的框架，线程会根据pipeline的需要自动创建和销毁，例如，将媒体流与应用线程解耦，应用线程不会被GStreamer的处理阻塞。而且，GStreaemer的插件还可以创建自己所需的线程用于媒体的处理，例如：在一个4核的CPU上，视频解码插件可以创建4个线程来最大化利用CPU资源。



此外，在创建pipeline时， 还可以指定某个pipeline 的分支在不同的线程中执行（例如，使video，audio同时在不同的线程中进行解码）。这是通过queue element实现的，queue的sink pad 仅仅将数据放入队列，另外一个线程从队列中取出数据，并传递到下一个element。queue 通常也被用于作为数据缓冲，缓冲区可以通过queue的属性进行配置。

![](C:\Users\hongmei.kuang\Pictures\typora\pthread.png)

在上面的示例Pipeline中，souce是audiotestsrc，会产生一个相应的audio信号，然后使用tee Element将数据分为两路，一路被用于播放，通过声卡输出，另一路被用于转换为视频波形，用于输出到屏幕。
示例图中的红色阴影部分表示位于同一个线程中，queue会创建单独的线程，所以上面的Pipeline使用了3个线程完成相应的功能。拥有多个sink的Pipeline通常需要多个线程，因为在多个sync间进行同步的时候，sink会阻塞当前所在线程直到所等待的事件发生。

```
#include <gst/gst.h>

int main(int argc, char *argv[]) {
  GstElement *pipeline, *audio_source, *tee, *audio_queue, *audio_convert, *audio_resample, *audio_sink;
  GstElement *video_queue, *visual, *video_convert, *video_sink;
  GstBus *bus;
  GstMessage *msg;
  GstPad *tee_audio_pad, *tee_video_pad;
  GstPad *queue_audio_pad, *queue_video_pad;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  audio_source = gst_element_factory_make ("audiotestsrc", "audio_source");
  tee = gst_element_factory_make ("tee", "tee");
  audio_queue = gst_element_factory_make ("queue", "audio_queue");
  audio_convert = gst_element_factory_make ("audioconvert", "audio_convert");
  audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
  audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
  video_queue = gst_element_factory_make ("queue", "video_queue");
  visual = gst_element_factory_make ("wavescope", "visual");
  video_convert = gst_element_factory_make ("videoconvert", "csp");
  video_sink = gst_element_factory_make ("autovideosink", "video_sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !audio_source || !tee || !audio_queue || !audio_convert || !audio_resample || !audio_sink ||
      !video_queue || !visual || !video_convert || !video_sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Configure elements */
  g_object_set (audio_source, "freq", 215.0f, NULL);
  g_object_set (visual, "shader", 0, "style", 1, NULL);

  /* Link all elements that can be automatically linked because they have "Always" pads */
  gst_bin_add_many (GST_BIN (pipeline), audio_source, tee, audio_queue, audio_convert, audio_resample, audio_sink,
      video_queue, visual, video_convert, video_sink, NULL);
  if (gst_element_link_many (audio_source, tee, NULL) != TRUE ||
      gst_element_link_many (audio_queue, audio_convert, audio_resample, audio_sink, NULL) != TRUE ||
      gst_element_link_many (video_queue, visual, video_convert, video_sink, NULL) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Manually link the Tee, which has "Request" pads */
  tee_audio_pad = gst_element_get_request_pad (tee, "src_%u");
  g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
  queue_audio_pad = gst_element_get_static_pad (audio_queue, "sink");
  tee_video_pad = gst_element_get_request_pad (tee, "src_%u");
  g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
  queue_video_pad = gst_element_get_static_pad (video_queue, "sink");
  if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK) {
    g_printerr ("Tee could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }
  gst_object_unref (queue_audio_pad);
  gst_object_unref (queue_video_pad);

  /* Start playing the pipeline */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* Release the request pads from the Tee, and unref them */
  gst_element_release_request_pad (tee, tee_audio_pad);
  gst_element_release_request_pad (tee, tee_video_pad);
  gst_object_unref (tee_audio_pad);
  gst_object_unref (tee_video_pad);

  /* Free resources */
  if (msg != NULL)
    gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);

  gst_object_unref (pipeline);
  return 0;
}
```

保存以上代码，执行下列编译命令即可得到可执行程序：

```
gcc basic-tutorial-8.c -o basic-tutorial-8 `pkg-config --cflags --libs gstreamer-1.0`
```

## 代码分析

```
/* Create the elements */
audio_source = gst_element_factory_make ("audiotestsrc", "audio_source");
tee = gst_element_factory_make ("tee", "tee");
audio_queue = gst_element_factory_make ("queue", "audio_queue");
audio_convert = gst_element_factory_make ("audioconvert", "audio_convert");
audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
video_queue = gst_element_factory_make ("queue", "video_queue");
visual = gst_element_factory_make ("wavescope", "visual");
video_convert = gst_element_factory_make ("videoconvert", "video_convert");
video_sink = gst_element_factory_make ("autovideosink", "video_sink");
```

首先创建所需的Element：audiotestsrc会产生测试的音频波形数据。wavescope 会将输入的音频数据转换为波形图像。audioconvert，audioresample，videoconvert保证了Pipeline中各个Element之间的数据可以互相兼容，使得Pipeline能够被正确的link起来，如果不需要对数据进行转换，这些Element会直接将数据发送到下一个Element，这种情况下的性能影响可以忽略不计。

```
/* Configure elements */
g_object_set (audio_source, "freq", 215.0f, NULL);
g_object_set (visual, "shader", 0, "style", 1, NULL);
```

这里修改相应Element的参数，使得输出结果更直观。“freq”会设置audiotestsrc输出波形的频率为215Hz，设置“shader”和“style”使得波形更加连续。其他的参数可以通过gst-inspect查看。

```
/* Link all elements that can be automatically linked because they have "Always" pads */
gst_bin_add_many (GST_BIN (pipeline), audio_source, tee, audio_queue, audio_convert, audio_sink,
    video_queue, visual, video_convert, video_sink, NULL);
if (gst_element_link_many (audio_source, tee, NULL) != TRUE ||
    gst_element_link_many (audio_queue, audio_convert, audio_sink, NULL) != TRUE ||
    gst_element_link_many (video_queue, visual, video_convert, video_sink, NULL) != TRUE) {
  g_printerr ("Elements could not be linked.\n");
  gst_object_unref (pipeline);
  return -1;
}
```

这里 使用gst_element_link_many 将多个Element连接起来，需要注意的是，这里 只连接了拥有Always Pad的Eelement。虽然gst_element_link_many() 能够在内部处理Request Pad的情况，但 仍然需要单独释放Request Pad，如果直接使用此函数连接所有的Element，这样容易忘记释放Request Pad。因此 使用下面的代码单独处理Request Pad。

```
/* Manually link the Tee, which has "Request" pads */
tee_audio_pad = gst_element_get_request_pad (tee, "src_%u");
g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
queue_audio_pad = gst_element_get_static_pad (audio_queue, "sink");
tee_video_pad = gst_element_get_request_pad (tee, "src_%u");
g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
queue_video_pad = gst_element_get_static_pad (video_queue, "sink");
if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
    gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK) {
  g_printerr ("Tee could not be linked.\n");
  gst_object_unref (pipeline);
  return -1;
}
gst_object_unref (queue_audio_pad);
gst_object_unref (queue_video_pad);
```

为了能够连接到Request Pad， 需要主动的向Element取得相应的Pad。由于一个Element可以提供不同的Request Pad，所以 需要指定所需的“Pad Template”，Element提供的Pad Template可以通过gst-inspect查看。从下面的结果可以发现，tee提供了2种类型的模板， ”sink“ 和“src_%u"。

```
$ gst-inspect-1.0  tee
...
Pad Templates:
  SRC template: 'src_%u'
    Availability: On request
      Has request_new_pad() function: gst_tee_request_new_pad
    Capabilities:
      ANY

  SINK template: 'sink'
    Availability: Always
    Capabilities:
      ANY
...
```

由于 这里需要的是2个Source Pad，所以 通过gst_element_get_request_pad (tee, "src_%u")获取两个Request Pad分别用于audio和video。queue的Sink Pad是Alwasy Pad，所以 直接使用gst_element_get_static_pad 获取其Sink Pad。最后再通过gst_pad_link()将其连接起来，在gst_element_link()和gst_element_link_many()内部也是使用此函数连接两个Element的Pad。

*需要注意的是， 通过Element获取到的Pad的引用计数会自动增加，因此 需要调用gst_object_unref()释放相关的引用，对于Request Pad， 需要在Pipeline执行完成后进行释放。*

```
/* Release the request pads from the Tee, and unref them */
gst_element_release_request_pad (tee, tee_audio_pad);
gst_element_release_request_pad (tee, tee_video_pad);
gst_object_unref (tee_audio_pad);
gst_object_unref (tee_video_pad);
```

除了播放完成后正常的资源释放外， 还要对Request进行释放，需要首先调用gst_element_release_request_pad()，最后再释放相应的对象。

# Appsrc 及Appsink

在前面所说的内容中，Pipline都是使用GStreamer自带的插件去产生/消费数据。在实际的情况中，数据源可能没有相应的gstreamer插件，但又需要将数据发送到GStreamer Pipeline中。GStreamer提供了Appsrc以及Appsink插件，用于处理这种情况，接下来将介绍如何使用这些插件来实现数据与应用程序的交互。

## Appsrc与Appsink

GStreamer提供了多种方法使得应用程序与GStreamer Pipeline之间可以进行数据交互，这里介绍的是最简单的一种方式：appsrc与appsink。

- appsrc：

用于将应用程序的数据发送到Pipeline中。应用程序负责数据的生成，并将其作为GstBuffer传输到Pipeline中。
appsrc有2中模式，拉模式和推模式。在拉模式下，appsrc会在需要数据时，通过指定接口从应用程序中获取相应数据。在推模式下，则需要由应用程序主动将数据推送到Pipeline中，应用程序可以指定在Pipeline的数据队列满时是否阻塞相应调用，或通过监听enough-data和need-data信号来控制数据的发送。

- appsink：

用于从Pipeline中提取数据，并发送到应用程序中。

appsrc和appsink需要通过特殊的API才能与Pipeline进行数据交互，相应的接口可以查看[官方文档](https://gstreamer.freedesktop.org/documentation/app/index.html?gi-language=c)，在编译的时候还需连接gstreamer-app库。

## GstBuffer

​	    在GStreamer Pipeline中的plugin间传输的数据块被称为buffer，在GStreamer内部对应于GstBuffer。Buffer由Source Pad产生，并由Sink Pad消耗。一个Buffer只表示一块数据，不同的buffer可能包含不同大小，不同时间长度的数据。同时，某些Element中可能对Buffer进行拆分或合并，所以GstBuffer中可能包含不止一个内存数据，实际的内存数据在GStreamer系统中通过GstMemory对象进行描述，因此，GstBuffer可以包含多个GstMemory对象。
​		每个GstBuffer都有相应的时间戳以及时间长度，用于描述这个buffer的解码时间以及显示时间。

建立如图所示pipeline的流程：

![](C:\Users\hongmei.kuang\Pictures\typora\create.png)

## 示例代码

![](C:\Users\hongmei.kuang\Pictures\typora\appsrc.png)

本例子在上一次的例子（多线程）的例子上进行扩展，首先使用appsrc 替代audiotestsrc用于产生audio数据，另外增加一个新的分支，将tee 产生的数据发送到应用程序，有应用程序决定如何处理收到的数据。

```
#include <gst/gst.h>
#include <gst/audio/audio.h>
#include <string.h>

#define CHUNK_SIZE 1024   /* Amount of bytes we are sending in each buffer */
#define SAMPLE_RATE 44100 /* Samples per second we are sending */

/* Structure to contain all our information, so we can pass it to callbacks */
typedef struct _CustomData {
  GstElement *pipeline, *app_source, *tee, *audio_queue, *audio_convert1, *audio_resample, *audio_sink;
  GstElement *video_queue, *audio_convert2, *visual, *video_convert, *video_sink;
  GstElement *app_queue, *app_sink;

  guint64 num_samples;   /* Number of samples generated so far (for timestamp generation) */
  gfloat a, b, c, d;     /* For waveform generation */

  guint sourceid;        /* To control the GSource */

  GMainLoop *main_loop;  /* GLib's Main Loop */
} CustomData;

/* This method is called by the idle GSource in the mainloop, to feed CHUNK_SIZE bytes into appsrc.
 * The idle handler is added to the mainloop when appsrc requests us to start sending data (need-data signal)
 * and is removed when appsrc has enough data (enough-data signal).
 */
static gboolean push_data (CustomData *data) {
  GstBuffer *buffer;
  GstFlowReturn ret;
  int i;
  GstMapInfo map;
  gint16 *raw;
  gint num_samples = CHUNK_SIZE / 2; /* Because each sample is 16 bits */
  gfloat freq;

  /* Create a new empty buffer */
  buffer = gst_buffer_new_and_alloc (CHUNK_SIZE);

  /* Set its timestamp and duration */
  GST_BUFFER_TIMESTAMP (buffer) = gst_util_uint64_scale (data->num_samples, GST_SECOND, SAMPLE_RATE);
  GST_BUFFER_DURATION (buffer) = gst_util_uint64_scale (num_samples, GST_SECOND, SAMPLE_RATE);

  /* Generate some psychodelic waveforms */
  gst_buffer_map (buffer, &map, GST_MAP_WRITE);
  raw = (gint16 *)map.data;
  data->c += data->d;
  data->d -= data->c / 1000;
  freq = 1100 + 1000 * data->d;
  for (i = 0; i < num_samples; i++) {
    data->a += data->b;
    data->b -= data->a / freq;
    raw[i] = (gint16)(500 * data->a);
  }
  gst_buffer_unmap (buffer, &map);
  data->num_samples += num_samples;

  /* Push the buffer into the appsrc */
  g_signal_emit_by_name (data->app_source, "push-buffer", buffer, &ret);

  /* Free the buffer now that we are done with it */
  gst_buffer_unref (buffer);

  if (ret != GST_FLOW_OK) {
    /* We got some error, stop sending data */
    return FALSE;
  }

  return TRUE;
}

/* This signal callback triggers when appsrc needs data. Here, we add an idle handler
 * to the mainloop to start pushing data into the appsrc */
static void start_feed (GstElement *source, guint size, CustomData *data) {
  if (data->sourceid == 0) {
    g_print ("Start feeding\n");
    data->sourceid = g_idle_add ((GSourceFunc) push_data, data);
  }
}

/* This callback triggers when appsrc has enough data and we can stop sending.
 * We remove the idle handler from the mainloop */
static void stop_feed (GstElement *source, CustomData *data) {
  if (data->sourceid != 0) {
    g_print ("Stop feeding\n");
    g_source_remove (data->sourceid);
    data->sourceid = 0;
  }
}

/* The appsink has received a buffer */
static GstFlowReturn new_sample (GstElement *sink, CustomData *data) {
  GstSample *sample;

  /* Retrieve the buffer */
  g_signal_emit_by_name (sink, "pull-sample", &sample);
  if (sample) {
    /* The only thing we do in this example is print a * to indicate a received buffer */
    g_print ("*");
    gst_sample_unref (sample);
    return GST_FLOW_OK;
  }

  return GST_FLOW_ERROR;
}

/* This function is called when an error message is posted on the bus */
static void error_cb (GstBus *bus, GstMessage *msg, CustomData *data) {
  GError *err;
  gchar *debug_info;

  /* Print error details on the screen */
  gst_message_parse_error (msg, &err, &debug_info);
  g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
  g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
  g_clear_error (&err);
  g_free (debug_info);

  g_main_loop_quit (data->main_loop);
}

int main(int argc, char *argv[]) {
  CustomData data;
  GstPad *tee_audio_pad, *tee_video_pad, *tee_app_pad;
  GstPad *queue_audio_pad, *queue_video_pad, *queue_app_pad;
  GstAudioInfo info;
  GstCaps *audio_caps;
  GstBus *bus;

  /* Initialize cumstom data structure */
  memset (&data, 0, sizeof (data));
  data.b = 1; /* For waveform generation */
  data.d = 1;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.app_source = gst_element_factory_make ("appsrc", "audio_source");
  data.tee = gst_element_factory_make ("tee", "tee");
  data.audio_queue = gst_element_factory_make ("queue", "audio_queue");
  data.audio_convert1 = gst_element_factory_make ("audioconvert", "audio_convert1");
  data.audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
  data.audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
  data.video_queue = gst_element_factory_make ("queue", "video_queue");
  data.audio_convert2 = gst_element_factory_make ("audioconvert", "audio_convert2");
  data.visual = gst_element_factory_make ("wavescope", "visual");
  data.video_convert = gst_element_factory_make ("videoconvert", "video_convert");
  data.video_sink = gst_element_factory_make ("autovideosink", "video_sink");
  data.app_queue = gst_element_factory_make ("queue", "app_queue");
  data.app_sink = gst_element_factory_make ("appsink", "app_sink");

  /* Create the empty pipeline */
  data.pipeline = gst_pipeline_new ("test-pipeline");

  if (!data.pipeline || !data.app_source || !data.tee || !data.audio_queue || !data.audio_convert1 ||
      !data.audio_resample || !data.audio_sink || !data.video_queue || !data.audio_convert2 || !data.visual ||
      !data.video_convert || !data.video_sink || !data.app_queue || !data.app_sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Configure wavescope */
  g_object_set (data.visual, "shader", 0, "style", 0, NULL);

  /* Configure appsrc */
  gst_audio_info_set_format (&info, GST_AUDIO_FORMAT_S16, SAMPLE_RATE, 1, NULL);
  audio_caps = gst_audio_info_to_caps (&info);
  g_object_set (data.app_source, "caps", audio_caps, "format", GST_FORMAT_TIME, NULL);
  g_signal_connect (data.app_source, "need-data", G_CALLBACK (start_feed), &data);
  g_signal_connect (data.app_source, "enough-data", G_CALLBACK (stop_feed), &data);

  /* Configure appsink */
  g_object_set (data.app_sink, "emit-signals", TRUE, "caps", audio_caps, NULL);
  g_signal_connect (data.app_sink, "new-sample", G_CALLBACK (new_sample), &data);
  gst_caps_unref (audio_caps);

  /* Link all elements that can be automatically linked because they have "Always" pads */
  gst_bin_add_many (GST_BIN (data.pipeline), data.app_source, data.tee, data.audio_queue, data.audio_convert1, data.audio_resample,
      data.audio_sink, data.video_queue, data.audio_convert2, data.visual, data.video_convert, data.video_sink, data.app_queue,
      data.app_sink, NULL);
  if (gst_element_link_many (data.app_source, data.tee, NULL) != TRUE ||
      gst_element_link_many (data.audio_queue, data.audio_convert1, data.audio_resample, data.audio_sink, NULL) != TRUE ||
      gst_element_link_many (data.video_queue, data.audio_convert2, data.visual, data.video_convert, data.video_sink, NULL) != TRUE ||
      gst_element_link_many (data.app_queue, data.app_sink, NULL) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Manually link the Tee, which has "Request" pads */
  tee_audio_pad = gst_element_get_request_pad (data.tee, "src_%u");
  g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
  queue_audio_pad = gst_element_get_static_pad (data.audio_queue, "sink");
  tee_video_pad = gst_element_get_request_pad (data.tee, "src_%u");
  g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
  queue_video_pad = gst_element_get_static_pad (data.video_queue, "sink");
  tee_app_pad = gst_element_get_request_pad (data.tee, "src_%u");
  g_print ("Obtained request pad %s for app branch.\n", gst_pad_get_name (tee_app_pad));
  queue_app_pad = gst_element_get_static_pad (data.app_queue, "sink");
  if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_app_pad, queue_app_pad) != GST_PAD_LINK_OK) {
    g_printerr ("Tee could not be linked\n");
    gst_object_unref (data.pipeline);
    return -1;
  }
  gst_object_unref (queue_audio_pad);
  gst_object_unref (queue_video_pad);
  gst_object_unref (queue_app_pad);

  /* Instruct the bus to emit signals for each received message, and connect to the interesting signals */
  bus = gst_element_get_bus (data.pipeline);
  gst_bus_add_signal_watch (bus);
  g_signal_connect (G_OBJECT (bus), "message::error", (GCallback)error_cb, &data);
  gst_object_unref (bus);

  /* Start playing the pipeline */
  gst_element_set_state (data.pipeline, GST_STATE_PLAYING);

  /* Create a GLib Main Loop and set it to run */
  data.main_loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (data.main_loop);

  /* Release the request pads from the Tee, and unref them */
  gst_element_release_request_pad (data.tee, tee_audio_pad);
  gst_element_release_request_pad (data.tee, tee_video_pad);
  gst_element_release_request_pad (data.tee, tee_app_pad);
  gst_object_unref (tee_audio_pad);
  gst_object_unref (tee_video_pad);
  gst_object_unref (tee_app_pad);

  /* Free resources */
  gst_element_set_state (data.pipeline, GST_STATE_NULL);
  gst_object_unref (data.pipeline);
  return 0;
}
```

编译

```
gcc basic-tutorial-9.c -o basic-tutorial-9 `pkg-config --cflags --libs gstreamer-1.0 gstreamer-audio-1.0 `
```

## 代码分析

首先对所需Element进行实例化，同时将Element的Always Pad连接起来，并与tee的Request Pad相连。此外还对appsrc及appsink进行了相应的配置：

```
/* Configure appsrc */
gst_audio_info_set_format (&info, GST_AUDIO_FORMAT_S16, SAMPLE_RATE, 1, NULL);
audio_caps = gst_audio_info_to_caps (&info);
g_object_set (data.app_source, "caps", audio_caps, NULL);
g_signal_connect (data.app_source, "need-data", G_CALLBACK (start_feed), &data);
g_signal_connect (data.app_source, "enough-data", G_CALLBACK (stop_feed), &data);
```

首先需要对appsrc的caps进行设定，指定会产生何种类型的数据，这样GStreamer会在连接阶段检查后续的Element是否支持此数据类型。这里的 caps必须为GstCaps对象，可以通过gst_caps_from_string()或gst_audio_info_to_caps ()得到相应的实例。
　　同时监听了“need-data”与“enough-data”事件，这2个事件由appsrc在需要数据和缓冲区满时触发，使用这2个事件可以方便的控制何时产生数据与停止数据。

```
/* Configure appsink */
g_object_set (data.app_sink, "emit-signals", TRUE, "caps", audio_caps, NULL);
g_signal_connect (data.app_sink, "new-sample", G_CALLBACK (new_sample), &data);
gst_caps_unref (audio_caps);
```

对于appsink，监听“new-sample”事件，用于appsink在收到数据时的处理。同时需要显式的使能“new-sample”事件，因为这个事件默认是处于关闭状态。

Pipeline的播放，停止及消息处理与其他示例相同，不再复述。接下来将查看监听事件的回调函数。

```
/* This signal callback triggers when appsrc needs data. Here, we add an idle handler
 * to the mainloop to start pushing data into the appsrc */
static void start_feed (GstElement *source, guint size, CustomData *data) {
  if (data->sourceid == 0) {
    g_print ("Start feeding\n");
    data->sourceid = g_idle_add ((GSourceFunc) push_data, data);
  }
}
```

appsrc会在其内部的数据队列即将缺乏数据时调用此回调函数，这里通过注册一个GLib的idle函数来向appsrc填充数据，GLib的主循环在“idle”状态时会循环调用 push_data，用于向appsrc填充数据。这只是一种向appsrc填充数据的方式， 可以在任意线程中想appsrc填充数据。
　　保存了g_idle_add()的返回值，以便后续用于停止数据写入。

```
/* This callback triggers when appsrc has enough data and we can stop sending.
 * We remove the idle handler from the mainloop */
static void stop_feed (GstElement *source, CustomData *data) {
  if (data->sourceid != 0) {
    g_print ("Stop feeding\n");
    g_source_remove (data->sourceid);
    data->sourceid = 0;
  }
}
```

stop_feed函数会在appsrc内部数据队列满时被调用。这里仅仅通过g_source_remove() 将先前注册的idle处理函数从GLib的主循环中移除（idle处理函数是被实现为一个GSource）。

```
/* This method is called by the idle GSource in the mainloop, to feed CHUNK_SIZE bytes into appsrc.
 * The ide handler is added to the mainloop when appsrc requests us to start sending data (need-data signal)
 * and is removed when appsrc has enough data (enough-data signal).
 */
static gboolean push_data (CustomData *data) {
  GstBuffer *buffer;
  GstFlowReturn ret;
  int i;
  gint16 *raw;
  gint num_samples = CHUNK_SIZE / 2; /* Because each sample is 16 bits */
  gfloat freq;

  /* Create a new empty buffer */
  buffer = gst_buffer_new_and_alloc (CHUNK_SIZE);

  /* Set its timestamp and duration */
  GST_BUFFER_TIMESTAMP (buffer) = gst_util_uint64_scale (data->num_samples, GST_SECOND, SAMPLE_RATE);
  GST_BUFFER_DURATION (buffer) = gst_util_uint64_scale (num_samples, GST_SECOND, SAMPLE_RATE);

  /* Generate some psychodelic waveforms */
  raw = (gint16 *)GST_BUFFER_DATA (buffer);
```

此函数会将真实的数据填充到appsrc的数据队列中，首先通过gst_buffer_new_and_alloc()分配一个GstBuffer对象，然后通过产生的采样数量计算这块buffre所对应的时间戳及事件长度。
　　gst_util_uint64_scale(val, num, denom)函数用于计算 val * num / denom，此函数内部会对数据范围进行检测，避免溢出的问题。
　　GstBuffer的数据指针可以通过GST_BUFFER_DATA 宏获取，在写数据时需要避免超出内存分配大小。本文将跳过audio波形生成的函数，其内容不是本文介绍的重点。

```
/* Push the buffer into the appsrc */
g_signal_emit_by_name (data->app_source, "push-buffer", buffer, &ret);

/* Free the buffer now that we are done with it */
gst_buffer_unref (buffer);
```

在准备好数据后，这里通过“push-buffer”事件通知appsrc数据就绪，并释放申请的buffer。 另外一种方式为通过调用[gst_app_src_push_buffer](https://developer.gnome.org/gst-plugins-libs/stable/gst-plugins-base-libs-appsrc.html#gst-app-src-push-buffer)() 向appsrc填充数据，这种方式就需要在编译时链接gstreamer-app-1.0库，同时gst_app_src_push_buffer() 会接管GstBuffer的所有权，调用者无需释放buffer。在所有数据都发送完成后，可以调用gst_app_src_end_of_stream()向Pipeline写入EOS事件。

```
/* The appsink has received a buffer */
static GstFlowReturn new_sample (GstElement *sink, CustomData *data) {
  GstSample *sample;
  /* Retrieve the buffer */
  g_signal_emit_by_name (sink, "pull-sample", &sample);
  if (sample) {
    /* The only thing we do in this example is print a * to indicate a received buffer */
    g_print ("*");
    gst_sample_unref (sample);
    return GST_FLOW_OK;
  }
  return GST_FLOW_ERROR;
}
```

当appsink得到数据时会调用new_sample函数，使用“pull-sample”信号提取sample，这里仅输出一个”*“表明此函数被调用。除此之外，同样可以使用gst_app_sink_pull_sample ()获取Sample。得到GstSample之后，可以通过gst_sample_get_buffer()得到Sample中所包含的GstBuffer，再使用GST_BUFFER_DATA， GST_BUFFER_SIZE 等接口访问其中的数据。使用完后，得到的GstSample同样需要通过gst_sample_unref()进行释放。
　　需要注意的是，在某些Pipeline里得到的GstBuffer可能会和source中填充的GstBuffer有所差异，因为Pipeline中的Element可能对Buffer进行各种处理（此例中不存在此种情况，因为在appsrc与appsink之间只存在一个tee）。

# streaming

把直接从网络播放一个媒体文件的方式称为在线播放（Online Streaming），在以往的例子中体验了GStreamer的在线播放功能，当我们指定播放URI为 http:// 时，GStreamer内部会自动通过网络获取媒体数据。在今天的示例中，将进一步了解如何处理由网络问题导致的视频缓冲及时钟丢失的问题。

## 在线播放

在进行在线播放时，会将收到的媒体数据立即进行解码并送入显示队列显示。当网络不理想时，通常不能及时的接收数据，显示队列中的数据会被耗尽而不能得到及时的补充，这会导致播放出现卡顿。
　　一种通用的处理方式是创建一个缓冲队列，在队列的数据量达到一定阀值时才进行播放，这样会导致起播时间会有一定的延迟，但会使后续的播放更加流畅，避免了因部分数据无法及时到达造成的停顿。
　　GStreamer框架已经实现了缓冲队列，但在以往的示例中并没有使用其相关的功能。某些Element（例如playbin中使用的queue2及multiqueue）可以创建缓冲队列，并在超过/低于指定的数据阀值时产生相应的信号。应用程序可以监听此类信号，在数据不足时（buffer值小于100%）主动暂停播放，在数据充足时恢复播放。
　　为了达到多个Sink的同步（例如音视频同步），需要使用一个全局的参考时钟，GStreamer会在播放时自动选取一个时钟。在某些网络在线播放的情况下原有时钟会失效，需要重新选取一个参考时钟。例如，RTP Source切换流或者改变输出设备。
　　在参考时钟丢失时，GStreamer框架会产生相应的事件，应用层需要对其作出响应，由于GStreamer在进入PLAYING状态时会自动选取参考时钟，所以只需在收到时钟丢失事件时将Pipeline的状态切换到PUASED，再切换到PLAYING即可。

## 示例代码

```
#include <gst/gst.h>
#include <string.h>

typedef struct _CustomData {
  gboolean is_live;
  GstElement *pipeline;
  GMainLoop *loop;
} CustomData;

static void cb_message (GstBus *bus, GstMessage *msg, CustomData *data) {

  switch (GST_MESSAGE_TYPE (msg)) {
    case GST_MESSAGE_ERROR: {
      GError *err;
      gchar *debug;

      gst_message_parse_error (msg, &err, &debug);
      g_print ("Error: %s\n", err->message);
      g_error_free (err);
      g_free (debug);

      gst_element_set_state (data->pipeline, GST_STATE_READY);
      g_main_loop_quit (data->loop);
      break;
    }
    case GST_MESSAGE_EOS:
      /* end-of-stream */
      gst_element_set_state (data->pipeline, GST_STATE_READY);
      g_main_loop_quit (data->loop);
      break;
    case GST_MESSAGE_BUFFERING: {
      gint percent = 0;

      /* If the stream is live, we do not care about buffering. */
      if (data->is_live) break;

      gst_message_parse_buffering (msg, &percent);
      g_print ("Buffering (%3d%%)\r", percent);
      /* Wait until buffering is complete before start/resume playing */
      if (percent < 100)
        gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
      else
        gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
      break;
    }
    case GST_MESSAGE_CLOCK_LOST:
      /* Get a new clock */
      gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
      gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
      break;
    default:
      /* Unhandled message */
      break;
    }
}

int main(int argc, char *argv[]) {
  GstElement *pipeline;
  GstBus *bus;
  GstStateChangeReturn ret;
  GMainLoop *main_loop;
  CustomData data;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Initialize our data structure */
  memset (&data, 0, sizeof (data));

  /* Build the pipeline */
  pipeline = gst_parse_launch ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);
  bus = gst_element_get_bus (pipeline);

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (pipeline);
    return -1;
  } else if (ret == GST_STATE_CHANGE_NO_PREROLL) {
    data.is_live = TRUE;
  }

  main_loop = g_main_loop_new (NULL, FALSE);
  data.loop = main_loop;
  data.pipeline = pipeline;

  gst_bus_add_signal_watch (bus);
  g_signal_connect (bus, "message", G_CALLBACK (cb_message), &data);

  g_main_loop_run (main_loop);

  /* Free resources */
  g_main_loop_unref (main_loop);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}
```

编译

```
gcc basic-tutorial-10.c -o basic-tutorial-10 `pkg-config --cflags --libs gstreamer-1.0`
```

由于通过网络获取数据，视频显示窗口可能会有短暂的等待时间，在终端的buffering达到100%时才会开始播放。

## 代码分析

GStreamer Pipeline相关的处理与以往示例相同，只关注在线播放相关的处理。

```
/* Start playing */
ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
if (ret == GST_STATE_CHANGE_FAILURE) {
  g_printerr ("Unable to set the pipeline to the playing state.\n");
  gst_object_unref (pipeline);
  return -1;
} else if (ret == GST_STATE_CHANGE_NO_PREROLL) {
  data.is_live = TRUE;
}
```

对于实时的媒体流，无法将其设置为PAUSED状态，所以在通过gst_element_set_state 将Pipeline设置成PAUSED状态时，会收到GST_STATE_CHANGE_NO_PREROLL。正常情况会返回GST_STATE_CHANGE_SUCCESS 。由于GStreamer的状态会依次从NULL, READY, PAUSED转换为PLAYING，所以将状态设置为PLAYING时，也会收到NO_PREROLL返回值。
　　这里设置is_live标识是因为不对其进行缓冲处理。

```
case GST_MESSAGE_BUFFERING: {
  gint percent = 0;

  /* If the stream is live, we do not care about buffering. */
  if (data->is_live) break;

  gst_message_parse_buffering (msg, &percent);
  g_print ("Buffering (%3d%%)\r", percent);
  /* Wait until buffering is complete before start/resume playing */
  if (percent < 100)
    gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
  else
    gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
  break;
}
```

在非实时流的情况下，如果缓存队列的数据不足，会收到GST_MESSAGE_BUFFERING事件，收到此事件时，可以通过gst_message_parse_buffering()得到缓冲进度，如果进度小于100%就暂停播放，在缓冲完成后再恢复播放。如果使用playbin，可以直接通过buffer-size或buffer-duration属性去修改缓冲区大小。

```
case GST_MESSAGE_CLOCK_LOST:
  /* Get a new clock */
  gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
  gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
  break;
```

针对于时钟丢失的这种情况，只需在收到GST_MESSAGE_CLOCK_LOST事件时，改变Pipline的状态，由GStreamer自动选取参考时钟即可。

# 常用命令工具

GStreamer 提供了不同的命令行工具用于快速的查看信息以及验证pipeline的是否能够正确运行，在平时的开发过程中优先使用GStreamer的命令行工具验证，再将pipeline集成到应用中。

## gst-inspect-1.0

此命令有三种工作模式，实际中常用第一种和第三种方式：

一、不带任何参数，这样会列出当前系统中支持的所有element ,这些element可用于构造pipeline.

二、跟文件名。这样会指定文件作为一个GStreamer插件，尝试列出其中所包含的element，例如下面的命令列出了libgstjpeg.so所包含的两个element。

```
$ gst-inspect-1.0 /usr/lib/x86_64-linux-gnu/gstreamer-1.0/libgstjpeg.so
Plugin Details:
  Name                     jpeg
  Description              JPeg plugin library
  Filename                 /usr/lib/x86_64-linux-gnu/gstreamer-1.0/libgstjpeg.so
  Version                  1.8.3
  License                  LGPL
  Source module            gst-plugins-good
  Source release date      2016-08-19
  Binary package           GStreamer Good Plugins (Ubuntu)
  Origin URL               https://launchpad.net/distros/ubuntu/+source/gst-plugins-good1.0

  jpegenc: JPEG image encoder
  jpegdec: JPEG image decoder

  2 features:
  +-- 2 elements
```

三、跟element名。会列出element的详细信息，例如，下面的命令会列出jpeg解码器所支持的输入数据类型，爬到信息，支持的属性及值，主要关注pad template 以及element的属性信息。

```
$ gst-inspect-1.0 jpegdec
Factory Details:
  Rank                     primary (256)
  Long-name                JPEG image decoder
  Klass                    Codec/Decoder/Image
  Description              Decode images from JPEG format
  Author                   Wim Taymans <wim@fluendo.com>

Plugin Details:
  Name                     jpeg
  Description              JPeg plugin library
  Filename                 /usr/lib/x86_64-linux-gnu/gstreamer-1.0/libgstjpeg.so
  Version                  1.8.3
  License                  LGPL
  Source module            gst-plugins-good
  Source release date      2016-08-19
  Binary package           GStreamer Good Plugins (Ubuntu)
  Origin URL               https://launchpad.net/distros/ubuntu/+source/gst-plugins-good1.0

GObject
 +----GInitiallyUnowned
       +----GstObject
             +----GstElement
                   +----GstVideoDecoder
                         +----GstJpegDec

Pad Templates:
  SINK template: 'sink'
    Availability: Always
    Capabilities:
      image/jpeg

  SRC template: 'src'
    Availability: Always
    Capabilities:
      video/x-raw
                 format: { I420, RGB, BGR, RGBx, xRGB, BGRx, xBGR, GRAY8 }
                  width: [ 1, 2147483647 ]
                 height: [ 1, 2147483647 ]
              framerate: [ 0/1, 2147483647/1 ]


Element Flags:
  no flags set

Element Implementation:
  Has change_state() function: gst_video_decoder_change_state

Element has no clocking capabilities.
Element has no indexing capabilities.
Element has no URI handling capabilities.

Pads:
  SINK: 'sink'
    Implementation:
      Has chainfunc(): gst_video_decoder_chain
      Has custom eventfunc(): gst_video_decoder_sink_event
      Has custom queryfunc(): gst_video_decoder_sink_query
      Has custom iterintlinkfunc(): gst_pad_iterate_internal_links_default
    Pad Template: 'sink'
  SRC: 'src'
    Implementation:
      Has custom eventfunc(): gst_video_decoder_src_event
      Has custom queryfunc(): gst_video_decoder_src_query
      Has custom iterintlinkfunc(): gst_pad_iterate_internal_links_default
    Pad Template: 'src'

Element Properties:
  name                : The name of the object
                        flags: readable, writable
                        String. Default: "jpegdec0"
  parent              : The parent of the object
                        flags: readable, writable
                        Object of type "GstObject"
  idct-method         : The IDCT algorithm to use
                        flags: readable, writable
                        Enum "GstIDCTMethod" Default: 1, "ifast"
                           (0): islow            - Slow but accurate integer algorithm
                           (1): ifast            - Faster, less accurate integer method
                           (2): float            - Floating-point: accurate, fast on fast HW
  max-errors          : (Deprecated) Error out after receiving N consecutive decoding errors (-1 = never fail, 0 = automatic, 1 = fail on first error)
                        flags: readable, writable, 0x80000000
                        Integer. Range: -1 - 2147483647 Default: 0
```

## gst-discoverer-1.0

是对GstDiscoverer接口的封装，可以方便查看媒体文件的编码，帧率等信息。

```
$ gst-discoverer-1.0 sintel_trailer-480p.mp4
Analyzing file:///home/xleng/video/sintel_trailer-480p.mp4
Done discovering file:///home/xleng/video/sintel_trailer-480p.mp4

Topology:
  container: Quicktime
    audio: MPEG-4 AAC
    video: H.264 (High Profile)

Properties:
  Duration: 0:00:52.209000000
  Seekable: yes
  Live: no
  Tags:
      audio codec: MPEG-4 AAC audio
      maximum bitrate: 128000
      datetime: 1970-01-01T00:00:00Z
      title: Sintel Trailer
      artist: Durian Open Movie Team
      copyright: (c) copyright Blender Foundation | durian.blender.org
      description: Trailer for the Sintel open movie project
      encoder: Lavf52.62.0
      container format: ISO MP4/M4A
      video codec: H.264 / AVC
      bitrate: 535929
```

## gst-launch0-1.0

gst-launch是平时使用最多的一个命令，它接收一个用字符串方式描述的Pipline，将其实例化并运行。可以用此命令快速的检查Pipeline中各个元素是否能够正确的连接起来。当需要构建的Pipeline很复杂时，也可以将Pipeline进行拆分，逐步通过gst-launch验证Pipeline的合法性。
　　通过gst-launch验证的字符串Pipeline可以直接使用gst_parse_launch()接口将其转化为GstPipeline对象，节省了我们单独调用API去创建Element的时间。
　　当我们用字符串描述Pipeline时，每个Element之间需要通过叹号 “!" 分隔Element，这样gst-launch才能正确识别。
　　在使用gst-launch时，根据不同的应用场景，可以分为以下的类型。

### 采用默认的参数创建pipeline

这种方式只需要将所使用的element使用叹号分隔即可

```
$ gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink
```

### 设置element的属性

在某些情况下，需要修改element的属性，或指定所需的参数（例如playbin的uri参数），element的属性直接跟在element之后，一下命令会设置videotestsrc的视频模式，输出图像为环形。属性支持的值可以通过gst-inspect命令查看。

```
$ gst-launch-1.0 videotestsrc pattern=11 ! videoconvert ! autovideosink
```

如果属性中包含空格，可以将其置于单引号或者双引号中。

```
$ gst-launch-1.0 -v videotestsrc ! clockoverlay halignment=right valignment=bottom text="Edge City" shaded-background=true font-desc="Sans, 36" ! videoconvert ! autovideosink
```

###  包含分支的pipeline

每个Element都有name的属性，可以利用name来实现包含多个分支的复杂的Pipeline，这常见于有多个输出/输入的Element（mux，demux，tee等）。
　　下面的命令通过tee创建了2个分支，分别用于不同的sink，在一个分支是Pipeline完成后（到达sink），可以使用“name加一个点号”重新创建一个分支。

```
$ gst-launch-1.0 videotestsrc ! videoconvert ! tee name=t ! queue ! autovideosink t. ! queue ! autovideosink
```

　使用同样的方式，我们也可以将多个分支合并为一个。下面的命令首先解码文件，将视频编码为H.264，音频编码为MP3，最终再合并生成TS文件。注意dmux和mux Element的使用。

```
$ gst-launch-1.0 filesrc location=surround.mp4 ! decodebin name=dmux ! queue ! audioconvert ! lamemp3enc ! mux. \
dmux. ! queue ! x264enc ! mpegtsmux name=mux ! queue ! filesink location=out.ts
```

### 指定element的pad

某些情况下，希望自己指定某个pad用于连接，可以指定已命名element的pad来实现。

```
$ gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! \
matroskademux name=d d.video_00 ! matroskamux ! filesink location=sintel_video.mkv
```

上面的命令会将sintel_trailer-480p.webm文件进行Demux，只选择其中的视频(video_00)，再将其保留为mk文件，只保留了视频部分。如果想只保留音频部分，可以使用如下命令：

```
$ gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! \
matroskademux name=d d.audio_00 ! vorbisparse ! matroskamux ! filesink location=sintel_audio.mka
```

上面两条命令均未对视频进行解码，只是将其从一个容器中拷入到另一个容器。

### 利用 caps filter 过滤码流

　当Element包含多个输出Pad时，可能导致连接到下一个Element的Pad具有不确定性。在下一个Element支持当前Element所有的输出类型，这时GStreamer会随机选择一个Pad用于连接。
　　例如：我们无法确定下面的Pipeline会使用video_00还是audio_00连接到filesink，因为filesink同时支持audio及video输入。

```
$ gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux ! filesink location=test
```

可以如上一节所说，显式指定连接所用的Pad，或者使用Caps Filter：

```
$ gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! \
matroskademux ! video/x-vp8 ! matroskamux ! filesink location=sintel_video.mkv
```

Caps Filter表现为一个只接收指定数据类型的Element，并将数据传递到下一个Element，可以高效的解决二义性的问题。
　　我们可以使用gst-inspect查看Element的输出Pad，以决定我们的Caps Filter需要添加何种参数。在gst-launch后加 -v 参数可以输出Pipeline连接时所使用的Pad信息。
　　对于多个过滤条件，我们需要通过逗号隔开。

　　对于某些特殊的类型参数，GStreamer提供了相应的关键字来进行转换：

- i 或 int 表示整型或范围；
- f 或 float 表示浮点数及范围；
- 4 或 fourcc 表示4个字符的FOURCC 代码；
- b, bool, 或 boolean 表示布尔型数据；
- s, str, 或 string 表示字符串；
- fraction 表示分数（例如帧率， 像素宽高比）；
- l 或 list 表示列表。

用于发送和接收UDP RTP数据：

```
$ gst-launch-1.0 v4l2src ! \
video/x-raw-yuv,width=128,height=96,format='(fourcc)'UYVY ! \
videoconvert ! ffenc_h263 ! video/x-h263 ! rtph263ppay pt=96 ! \
udpsink host=192.168.1.1 port=5000 sync=false

$gst-launch-1.0 udpsrc port=5000 ! application/x-rtp, \
clock-rate=90000,payload=96 ! rtph263pdepay queue-delay=0 ! ffdec_h263 \
! xvimagesink
```

使用YUY2或YV12作为测试视频格式，帧率为30帧/秒：

```
$ gst-launch-1.0 videotestsrc ! \
'video/x-raw-yuv,format=(fourcc)YUY2,framerate=30/1;video/x-raw-yuv,format=(fourcc)YV12,framerate=30/1' \
! xvimagesink
```

通过alsasrc录制文件，限定采样率及位宽：

```
$ gst-launch-1.0 alsasrc! \
'audio/x-raw-int,rate=[32000,64000],width=[16,32],depth={16,24,32},signed=(boolean)true' \
! wavenc ! filesink location=recording.wav
```

# 调试 pipeline 

在很多情况下，我们需要对GStreamer创建的Pipeline进行调试，来了解其运行机制以解决所遇到的问题。为此，GStreamer提供了相应的调试机制，方便快速定位问题。

## 查看调试日志

### 使用GST_DEBUG环境变量查看日志

GStreamer框架以及其插件提供了不同级别的日志信息，日志中包含时间戳，进程ID，线程ID，类型，源码行数，函数名，Element信息以及相应的日志消息。例如：

```
$ GST_DEBUG=2 gst-launch-1.0 playbin uri=file:///x.mp3
Setting pipeline to PAUSED ...
0:00:00.014898047 47333      0x2159d80 WARN                 filesrc gstfilesrc.c:530:gst_file_src_start:<source> error: No such file "/x.mp3"
...
```

我们可以发现，只需要在运行时指定GST_DEBUG环境变量，并设置日志级别，即可得到相应的日志。由于GStreamer提供了丰富的日志，如果我们打开所有的日志，必定会对程序的性能有所影响，所以我们需要对日志进行分级，GStreamer提供了8种级别，用于输出不同类型的日志。

- 级别0：不输出任何日志信息。
- 级别1：ERROR信息。
- 级别2：WARNING信息。
- 级别3：FIXME信息。
- 级别4：INFO信息。
- 级别5：DEBUG信息
- 级别6：LOG信息。
- 级别7：TRACE信息。
- 级别8：MEMDUMP信息，最高级别日志。

　　在使用时，我们只需将GST_DEBUG设置为相应级别，所有小于其级别的信息都会被输出，例如：设置GST_DEBUG=2，我们会得到ERROR及WARNING级别的日志。
　　上面的例子中，所有模块使用同一日志级别，除此之外，我们还可以针对某个插件设定其独有的日志级别，例如：GST_DEBUG=2,audiotestsrc:6 只会将audiotestsrc的日志级别设置为6，其他的模块仍然使用级别2。
　　这样，GST_DEBUG的值是以逗号分隔的”模块名:级别“的键值对，可以在最开始增加其他未指定模块的默认日志级别，多个模块名可以使用逗号隔开。同时，GST_DEBUG的值还支持”*“通配符。
　　例如：GST_DEBUG=2,audio*:6会将模块名以audio开始的模块的日志级别设置为6，其他的默认为2。
　　同样，GST_DEBUG=*:2 会匹配所有的模块，与GST_DEBUG=2等同。
　　我们可以通过gst-launch-1.0 --gst-debug-help 列出当前所注册的模块名，模块名由插件注册。在安装的插件改变时，此命令输出结果也会变化。

**在Windows中，需要按照Windows的方式设置环境变量**。

- 　　Windows cmd使用以下命令设置：

```
D:\gstreamer\1.0\msvc_x86_64\bin>  set GST_DEBUG=*:2

查看设置的值
D:\gstreamer\1.0\msvc_x86_64\bin>  set GST_DEBUG
GST_DEBUG=*:2
```

- 　　Windows PowerShell使用以下命令设置：

```
PS D:\Tools\gstreamer\1.0\msvc_x86_64\bin> $env:GST_DEBUG="*:2"

查看设置的值
PS D:\Tools\gstreamer\1.0\msvc_x86_64\bin> $env:GST_DEBUG
*:2

PS D:\Tools\gstreamer\1.0\msvc_x86_64\bin> $env:GST_DEBUG="souphttpsrc:6"
PS D:\Tools\gstreamer\1.0\msvc_x86_64\bin> $env:GST_DEBUG
souphttpsrc:6
```

### 使用GST_DEBUG_FILE将日志输出到文件

在实际中，我们通常将日志保存在文件中，便于后续分析。我们可以使用GST_DEBUG_FILE环境变量，指定日志文件名，GStreamer会自动将日志写入文件中，由于GStreamer日志包含终端色彩代码，我们通常使用 GST_DEBUG_NO_COLOR=1 环境变量将其禁用，方便查看。使用方式如下：

```
$ GST_DEBUG_NO_COLOR=1 GST_DEBUG_FILE=pipeline.log GST_DEBUG=5 gst-launch-1.0 audiotestsrc ! autoaudiosink
```

### 使用自定义日志接口

在实际项目中，不同应用可能采用不同的日志接口，为此，GStreamer提供了相应的接口，应用程序可以在初始化时，通过[gst_debug_add_log_function](https://gstreamer.freedesktop.org/documentation/gstreamer/gstinfo.html?gi-language=c)()增加自定义日志接口。相关接口如下：

```
//Add customized log function to GStreamer log system.
void gst_debug_add_log_function (GstLogFunction func,
                            gpointer user_data,
                            GDestroyNotify notify);

// Function prototype for a logging function that can be registered with
// gst_debug_add_log_function()
// Use G_GNUC_NO_INSTRUMENT on that function.
typedef void (*GstLogFunction) (GstDebugCategory * category,
                   GstDebugLevel level,
                   const gchar * file,
                   const gchar * function,
                   gint line,
                   GObject * object,
                   GstDebugMessage * message,
                   gpointer user_data);

// Enable log if set to true.
void gst_debug_set_active (gboolean active);
// Set the default log level.
void gst_debug_set_default_threshold (GstDebugLevel level);
```

示例代码如下

```
#include <gst/gst.h>
#include <stdio.h>

/* declare log function with the required attribute */
void my_log_func(GstDebugCategory * category,
                 GstDebugLevel level,
                 const gchar * file,
                 const gchar * function,
                 gint line,
                 GObject * object,
                 GstDebugMessage * message,
                 gpointer user_data) G_GNUC_NO_INSTRUMENT;

void my_log_func(GstDebugCategory * category,
                 GstDebugLevel level,
                 const gchar * file,
                 const gchar * function,
                 gint line,
                 GObject * object,
                 GstDebugMessage * message,
                 gpointer user_data) {

    printf("MyLogFunc: [Level:%d] %s:%s:%d  %s\n",
            level, file, function, line,
            gst_debug_message_get(message));

}

int main(int argc, char *argv[]) {
  GstPipeline *pipeline = NULL;
  GMainLoop *main_loop = NULL;

  /* set log function and remove the default one */
  gst_debug_add_log_function(my_log_func, NULL, NULL);
  gst_debug_set_active(TRUE);
  gst_debug_set_default_threshold(GST_LEVEL_INFO);

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* default log function is added by gst_init, so we need remove it after that. */
  gst_debug_remove_log_function(gst_debug_log_default);


  pipeline = (GstPipeline *)gst_parse_launch("audiotestsrc ! autoaudiosink", NULL);

  /* Start playing */
  gst_element_set_state (GST_ELEMENT(pipeline), GST_STATE_PLAYING);

  main_loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (main_loop);

  /* Free resources */
  g_main_loop_unref (main_loop);
  gst_element_set_state (GST_ELEMENT(pipeline), GST_STATE_NULL);
  gst_object_unref (pipeline);

  return 0;
}
```

编译运行后，会得到指定函数输出的log。

```
$ gcc basic-tutorial-13a.c -o basic-tutorial-13a `pkg-config --cflags --libs gstreamer-1.0`
```

### 使用GStreamer日志系统

如果不想使用自定义接口，我们同样可以使用Gstreamer提供的日志系统来由Gstreamer框架统一管理日志。
　　使用GStreamer的日志系统时，我们需要首先定义我们的category，并定义GST_CAT_DEFAULT 宏为我们的category：

```
GST_DEBUG_CATEGORY_STATIC (my_category);
#define GST_CAT_DEFAULT my_category
```

然后在gst_init后初始化我们的category：

```
GST_DEBUG_CATEGORY_INIT (my_category, "my category", 0, "This is my very own");
```

最后使用GST_ERROR(), GST_WARNING(), GST_INFO(), GST_LOG() 或GST_DEBUG() 宏输出日志，这些宏所接受的参数类型与printf相同。
　　示例代码如下：

```
#include <gst/gst.h>
#include <stdio.h>

GST_DEBUG_CATEGORY_STATIC (my_category);
#define GST_CAT_DEFAULT my_category

int main(int argc, char *argv[]) {
  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  GST_DEBUG_CATEGORY_INIT (my_category, "my category", 0, "This is my very own");

  GST_ERROR("My msg: %d", 0);
  GST_WARNING("My msg: %d", 1);
  GST_INFO("My msg: %d", 2);
  GST_DEBUG("My msg: %d", 3);

  return 0;
}
```

编译后，设置相应的log等级即可看到我们所添加的log。

```
$ gcc basic-tutorial-13b.c -o basic-tutorial-13b `pkg-config --cflags --libs gstreamer-1.0`
$ GST_DEBUG=5 ./basic-tutorial-13b
...
0:00:00.135957434  6189      0x21c4600 ERROR            my category basic-tutorial-13b.c:13:main: My msg: 0
0:00:00.135967528  6189      0x21c4600 WARN             my category basic-tutorial-13b.c:14:main: My msg: 1
0:00:00.135976899  6189      0x21c4600 INFO             my category basic-tutorial-13b.c:15:main: My msg: 2
0:00:00.135985622  6189      0x21c4600 DEBUG            my category basic-tutorial-13b.c:16:main: My msg: 3
```

### 获取Pipeline运行时的Element关系图

在Pipeline变得很复杂时，我们需要知道Pipeline是否按预期运行、使用到哪些Element，尤其是使用playbin 或uridecodebin时。为此，GStreamer提供了相应的功能，能够将Pipeline在当前状态下所有的Elements及其关系输出成dot文件，再通过 Graphviz等工具可以将其转换成图片文件。
　　为了得到.dot文件，我们只需通过GST_DEBUG_DUMP_DOT_DIR 环境变量，指定输出目录即可，gst-launch-1.0会在各状态分别生成一个.dot文件。 例如：通过下列命令，我们可以得到使用playbin播放网络文件时生成的Pipeline：

```
$ GST_DEBUG_DUMP_DOT_DIR=. gst-launch-1.0 playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm
$ ls *.dot
0.00.00.013715494-gst-launch.NULL_READY.dot    
0.00.00.170999259-gst-launch.PAUSED_PLAYING.dot  
0.00.07.642049256-gst-launch.PAUSED_READY.dot
0.00.00.162033239-gst-launch.READY_PAUSED.dot  
0.00.07.606477348-gst-launch.PLAYING_PAUSED.dot

$ dot 0.00.00.170999259-gst-launch.PAUSED_PLAYING.dot -Tpng -o play.png
```

结果会根据安装的插件不同而不同

**需要注意的是，如果需要在自己的应用中加入此功能，那就需要在想要生成dot文件的时候显式地在相应事件发生时调用[GST_DEBUG_BIN_TO_DOT_FILE](https://developer.gnome.org/gstreamer/stable/gstreamer-GstInfo.html#GST-DEBUG-BIN-TO-DOT-FILE:CAPS)() 或[GST_DEBUG_BIN_TO_DOT_FILE_WITH_TS](https://developer.gnome.org/gstreamer/stable/gstreamer-GstInfo.html#GST-DEBUG-BIN-TO-DOT-FILE-WITH-TS:CAPS)()，否则即使设置了GST_DEBUG_DUMP_DOT_DIR 环境变量也无法生成dot文件。**

 
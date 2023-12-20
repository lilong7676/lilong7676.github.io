---
title: mytoolbox - 有趣的想法之实现一个粘贴板历史管理工具
date: 2023-11-21 18:09:07
tags:
categories: flutter
---

windows 上好像已经有了自带的粘贴板历史工具，但是 mac 上没有自带的。之前一直在使用一个叫 pastebot 的工具，后面不支持 mac 新系统了（我的是 13.2.1），然后又找到一个叫 Paste 的工具，但是是收费的。
我觉得这种软件实现起来应该很简单吧，所以不如动手自己做一个。

## 技术选型
考虑到主要是桌面端使用，首先想到的就是 Electron 了，之前也做过相关开发，对 web 前端开发者算是很友好了。但是总觉得 Electron 实现的效果不够优雅（总觉得太像网页了哈哈）。
想了一下，最终选择了 Flutter，可以同时支持桌面端和移动端（因为我后面还想做一些小工具，比如私人GPT小助手，想支持跨端）。
哈哈，我的想法可太多了。


<!-- more -->


## 实现效果
先来看看最终实现的效果吧
![粘贴板历史管理器实现效果](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/202312201038188.png?x-oss-process=image/resize,w_800)

![点击历史项就可以复制到粘贴板](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/202312201039776.png?x-oss-process=image/resize,w_800)

## 实现思路
其实本质上就是通过监听系统粘贴板事件，然后取出里面的内容，通过 Flutter Listview 展示出来。做一些简单的交互即可。
这其中用到了一些库，主要有：
- [clipboard_watcher](https://pub.dev/packages/clipboard_watcher)，用来监听系统粘贴板事件
- [super_clipboard](https://pub.dev/packages/super_clipboard)，用来读取和写入到粘贴板
- [super_hot_key](https://pub.dev/packages/super_hot_key)，用来定义系统级全局快捷键

## 关键代码
[mytoolbox仓库地址](https://github.com/lilong7676/mytoolbox)  

```dart
import 'dart:async';
import 'package:flutter/services.dart';
import 'package:flutter/material.dart';
import 'package:clipboard_watcher/clipboard_watcher.dart';
import 'package:super_clipboard/super_clipboard.dart';

class Paster extends StatefulWidget {
  const Paster({Key? key}) : super(key: key);

  @override
  PasterState createState() => PasterState();
}

class PasterState extends State<Paster> with ClipboardListener {
  List<ClipboardHistoryItem> historyList = [];

  bool tempDisableListener = false;

  @override
  void initState() {
    // 监听粘贴板
    clipboardWatcher.addListener(this);
    // start watch
    clipboardWatcher.start();
    super.initState();
  }

  @override
  void dispose() {
    clipboardWatcher.removeListener(this);
    // stop watch
    clipboardWatcher.stop();
    super.dispose();
  }

  @override
  void onClipboardChanged() async {
    if (tempDisableListener) {
      return;
    }

    final item = await ClipboardHistoryItem.readFromClipboard();
    if (item.contentType == ClipboardContentType.empty) {
      return;
    }
    setState(() {
      historyList.insert(0, item);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Padding(padding: EdgeInsets.all(8), child: Text('粘贴板历史')),
        const Divider(),
        Flexible(
            child: Align(
                alignment: Alignment.topLeft,
                child: _buildClipboardHistoryList(context))),
      ],
    );
  }

  Widget _buildClipboardHistoryList(BuildContext context) {
    // 横向滚动
    return ConstrainedBox(
        constraints: const BoxConstraints(maxHeight: 300),
        child: ListView.separated(
          padding: const EdgeInsets.all(8),
          scrollDirection: Axis.horizontal,
          itemCount: historyList.length,
          itemBuilder: (BuildContext context, int index) {
            final historyItem = historyList[index];

            late Widget title;
            switch (historyItem.contentType) {
              case ClipboardContentType.text:
                title = Text(historyItem.text!);
                break;
              case ClipboardContentType.image:
                title = Image.memory(historyItem.imageBytes!);
                break;
              default:
                title = const Text('未知类型');
            }

            return Container(
                width: 280,
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                    border: Border.all(color: Colors.grey),
                    borderRadius: BorderRadius.circular(8)),
                child: Column(
                  children: [
                    Row(
                      children: [
                        Expanded(
                            child: Text(
                                '${historyItem.timestamp!.hour}:${historyItem.timestamp!.minute}:${historyItem.timestamp!.second}')),
                        IconButton(
                            onPressed: () {
                              setState(() {
                                historyList.removeAt(index);
                              });
                            },
                            icon: const Icon(Icons.delete))
                      ],
                    ),
                    const Divider(),
                    Expanded(
                        child: ListTile(
                            title: title,
                            onTap: () async {
                              // 临时禁用监听
                              tempDisableListener = true;
                              // 写入剪贴板
                              await historyItem.writeToClipboard();
                              // TODO 待解决：理想情况下，首先需要使应用进入后台，然后激活之前的应用（有输入框的应用）
                              // 然后进行粘贴操作
                              // await FlutterClipboard.paste();
                              // 延迟后再次启用监听
                              Future.delayed(const Duration(milliseconds: 100),
                                  () {
                                tempDisableListener = false;
                                // 并将当前的历史记录移动到第一位
                                setState(() {
                                  historyList.removeAt(index);
                                  historyList.insert(0, historyItem);
                                });
                              });

                              // 展示提示信息
                              const snackBar = SnackBar(
                                content: Text('已复制到粘贴板～'),
                                duration: Duration(seconds: 2),
                                showCloseIcon: true,
                              );
                              if (!context.mounted) return;
                              ScaffoldMessenger.of(context)
                                  .showSnackBar(snackBar);
                            }))
                  ],
                ));
          },
          separatorBuilder: (BuildContext context, int index) =>
              const VerticalDivider(width: 10, color: Colors.transparent),
        ));
  }
}

enum ClipboardContentType { empty, text, image }

class ClipboardHistoryItem {
  String? text;
  Uint8List? imageBytes;
  DateTime? timestamp;

  late ClipboardContentType contentType;

  ClipboardHistoryItem({this.text, this.imageBytes}) {
    if (text != null) {
      contentType = ClipboardContentType.text;
    } else if (imageBytes != null) {
      contentType = ClipboardContentType.image;
    } else {
      contentType = ClipboardContentType.empty;
    }
    timestamp = DateTime.now();
  }

  Future<void> writeToClipboard() async {
    final item = DataWriterItem();
    if (imageBytes != null) {
      item.add(Formats.png(imageBytes!));
    }
    if (text != null) {
      item.add(Formats.plainText(text!));
    }
    await ClipboardWriter.instance.write([item]);
  }

  static Future<ClipboardHistoryItem> readFromClipboard() async {
    final reader = await ClipboardReader.readClipboard();

    if (reader.canProvide(Formats.plainText)) {
      final text = await reader.readValue(Formats.plainText);
      return ClipboardHistoryItem(text: text);
    } else if (reader.canProvide(Formats.png)) {
      // 暂时只支持 png file
      final imageBytes =
          await ClipboardHistoryItem.readFile(reader, Formats.png);
      return ClipboardHistoryItem(imageBytes: imageBytes);
    } else {
      return ClipboardHistoryItem();
    }
  }

  static Future<Uint8List?>? readFile(
      ClipboardReader reader, FileFormat format) {
    final c = Completer<Uint8List?>();
    final progress = reader.getFile(format, (file) async {
      try {
        final all = await file.readAll();
        c.complete(all);
      } catch (e) {
        c.completeError(e);
      }
    }, onError: (e) {
      c.completeError(e);
    });
    if (progress == null) {
      c.complete(null);
    }
    return c.future;
  }
}

```


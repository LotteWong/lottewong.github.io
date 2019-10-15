---
title: \#Configuration\# HTTP的通信监控和回放：Fiddler抓包与Socket发包
date: 2019-10-11 07:00:21
categories: Testing
tags:
- Fiddler
- Curl
- Socket
thumbnail: /images/fiddler.png
---





利用Fiddler抓包和Socket发包，并用Curl脚本统计响应时间，主要分为以下四个部分：

- 工具的配置及工具的使用
- **截获、修改、发送数据包**
- **统计网页和元素的响应时间**
- 需注意事项及待改进事项



<!-- more -->

## **工具的配置**

### **安装工具**

[Fiddler下载地址](https://www.telerik.com/fiddler)

### **导入证书**

![Decrypt to capture Https](/images/decrypt_https_1.png)

![Decrypt to capture Https](/images/decrypt_https_2.png)

### **勾选解码**

![GZip encoding is default in Fiddler](/images/encode_decode.png)

### **过滤保存**

#### **手动**

![Set filters](/images/filters.png)

![Save sessions](/images/save_sessions.png)

#### **脚本**

![Customize Rules](/images/customize_rules_1.png)

![Edit scripts](/images/customize_rules_2.png)

```javascript
if (oSession.fullUrl.Contains("baidu.com")) {
    // for(var key in oSession.oRequest.headers) {
        // if('Referer' === key) {
            if(oSession.oRequest.headers['Referer'].indexOf("&wd=") != -1) {
                var fso;
                var file;
                fso = new ActiveXObject("Scripting.FileSystemObject");
                file = fso.OpenTextFile("E:\\MyPrograms\\fiddler_sessions\\Session" + new Date().getTime() + ".txt", 8 ,true, true);
                // file.writeLine("Request url: " + oSession.url);
                file.writeLine("Request header:" + "\n" + oSession.oRequest.headers);
                // file.writeLine("Request body: " + oSession.GetRequestBodyAsString());
                file.writeLine("\n");
                file.close();
            }
        // }
    // }
}
```

### **设置断点**

#### **手动**

![Set Breakpoints](/images/breakpoints.png)

#### **命令**

- 在左下角黑框框中输入命令
  - 停止断点：`bpu `
  - 开始断点：`bpu $host`

## **工具的使用**

### **Statistics**

![Statistics](/images/statistics.png)

### **Filters**

![Filters](/images/filters.png)

### **Inspectors**

![Inspectors](/images/inspectors.png)

### **Composer**

![Composer](/images/composer.png)

### **AutoResponder**

![AutoResponder](/images/auto_responder.png)

## **截获数据包**

### **图形界面**

- 使用图形界面或编程脚本应用过滤
- 点击左下角 `Capturing或空白处` 停止或开始截获数据包
- 使用Statistics查看时间，使用Inspectors查看内容
- 使用图形界面或编程脚本保存会话

## **修改数据包**

### **程序脚本**

#### **对于手动保存的会话**

  ``` python
  # 替换请求中的搜索字段
  def get_ref(file, cont):
      with open(file, encoding='utf-8') as f:
          lines = f.readlines()
          for line in lines:
              if 'Referer' in line:
                  start = line.find("&wd=")
                  end = line.find("&rsv_pq=")
                  old_str = line[start+4:end]
                  new_str = parse.quote(cont)
                  line = line.replace(old_str, new_str)
                  return line
  
  
  # 重新拼装需发送的报文
  def get_req(file, cont):
      msg = ''
      with open(file, encoding='utf-8') as f:
          lines = f.readlines()
          for line in lines:
              if 'GET' in line:
                  line = 'GET ' + get_ref(file, cont)[9:-1] + ' HTTP/1.1'
              if 'Referer' in line:
                  line = get_ref(file, cont)
              if 'Accept-Encoding' in line:
                  continue
              line = line.strip('\n') + '\r\n'
              msg += line
          msg = bytes(msg, encoding="utf8")
          return msg
  ```

#### **对于自动保存的会话**

  ``` python
  # 替换请求中的搜索字段
  def get_ref(file, cont):
      with open(file, encoding='utf-16') as f:
          lines = f.readlines()
          for line in lines:
              if 'Referer' in line:
                  start = line.find("&wd=")
                  end = line.find("&rsv_pq=")
                  old_str = line[start+4:end]
                  new_str = parse.quote(cont)
                  line = line.replace(old_str, new_str)
                  return line
  
  
  # 重新拼装需发送的报文
  def get_req(file, cont):
      msg = ''
      with open(file, encoding='utf-16') as f:
          lines = f.readlines()
          for line in lines:
              if 'Request header:' in line:
                  continue
              if 'GET' in line:
                  line = 'GET ' + get_ref(file, cont)[9:-1] + ' HTTP/1.1'
              if 'Referer' in line:
                  line = get_ref(file, cont)
              if 'Accept-Encoding' in line:
                  continue
              line = line.strip('\n') + '\r\n'
              msg += line
          msg = bytes(msg[:-4], encoding="utf8")
          return msg
  ```

## **发送数据包**

### **Fiddler**

#### **使用WebForms**

- 使用图形界面或编程脚本应用过滤
- 使用图形界面或运行命令设置断点
- 在 `对应报文A` 的 `Request WebForms`内修改“搜索关键字”
- 点击 `Break on Response` 将修改后的 `对应报文A` 发送到Fiddler
- 在 `对应报文B` 的 `Request WebForms`内查看“搜索关键字”
- 点击 `Run to Completion` 将修改后的 `对应报文B` 发送到Server

#### **使用Composer**

- 保存会话Request请求头部
- 在记事本内修改“搜索关键字”
- 在Composer内发送请求报文

### **Socket**

  ``` python
  if __name__ == '__main__':
      # 读取信息
      file = input("file:")
      cont = input("cont:")
  
      # 套接字连接服务端
      s = ssl.wrap_socket(socket.socket())
      s.connect(('www.baidu.com', 443))
  
      # 发送修改后的请求
      s.send(get_req(file, cont))
  
      # 缓存服务端的响应
      buffer = []
      while True:
          d = s.recv(1024)
          if d:
              buffer.append(d)
          else:
              break
      res = b''.join(buffer)
  
      # 客户端关闭套接字
      s.close()
  
      # 保存响应
      header, html = res.split(b'\r\n\r\n', 1)
      print(header.decode('utf-8'))
      with open(cont + '.html', 'wb') as f:
          f.write(html)
  ```

## **记响应时间**

### **整个网页**

- To be continue...

### **单个元素**

- To be continue...

## **实例的演示**

![Flowchart](/images/fiddler_socket.png)

- [仅使用Fiddler抓包、修改、发包](录屏链接To be continue...)
- [Fiddler抓包+Python修改+Socket发包](录屏链接To be continue...)

## **需注意事项**

### **报文格式**

- 无论使手动还是脚本保存会话的请求报文，都需要注意每个属性是否以 `\r\n`  结尾，最后属性是否以 `\r\n\r\n` 结尾
- 遇到 `HTTP 400 Bad Request` 响应仔细检查报文格式是否正确

### **编码问题**

- 注意保存会话的编码格式，手动保存使用编码格式 `utf-8` ，脚本保存使用编码格式 `utf-16`
- Socket发送报文和接受报文都需要二进制数据
- Fiddler默认使用GZip格式压缩，在发送请求报文时为确保响应主题非乱码，应该去除 `Accept-Encoding: gzip, deflate` 这行属性

### **端口问题**

- Socket通信需要知道主机地址及其端口号
  - Fiddler Sessions或Inpectors可知主机地址及其端口号
  - 保存的TCP报文（使用Wireshark）可知主机地址及其端口号
  - 保存的HTTP/HTTPS报文（使用Fiddler）仅知主机地址，已知常用端口：HTTP为80/HTTPS为443

### **请求变化**

- `Break on Response` 和 `Run to Completion` 对应会话并不相同

## **待改进事项**

- 优化过滤会话和替换内容脚本（正则表达式）

- 发送响应回浏览器（Socket向其它进程发报文）

- 持续化、多线程抓包、修改、发包（多线程编程）
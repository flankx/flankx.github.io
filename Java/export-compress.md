# 导出文件压缩

## 1 导出文件

+ 1.1 设置返回的请求头

| Response Header     | Value                        | Desc                                                                                      |
| ------------------- | ---------------------------- | ----------------------------------------------------------------------------------------- |
| Content-type        | application/json             | [资源的MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)  |
| Content-Disposition | attachment;filename=xxx.json | [展示形式](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition) |

+ 1.2 获取请求导出的资源并写入输出流

````java
response.setCharacterEncoding("UTF-8");
response.setContentType("application/json");
response.addHeader("Content-Disposition", "attachment;filename=xxx.json");
    
String content = ...; // 此处导出字符串
try (ServletOutputStream outStr = response.getOutputStream();
    BufferedOutputStream buff = new BufferedOutputStream(outStr)) {
    buff.write(content.getBytes(StandardCharsets.UTF_8));
    buff.flush();
} catch (Exception e) {
    e.printStackTrace();
}
````

## 2 导出文件并压缩

### 2.1 使用压缩工具`Apache Commons Compress`

+ 引入`POM`依赖

````xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-compress -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.22</version>
</dependency>
````

+ 加入`Zip`压缩

````java
ZipArchiveOutputStream zipOutput = ...;
ZipArchiveEntry entry = new ZipArchiveEntry(name);
entry.setSize(size);
zipOutput.putArchiveEntry(entry);
zipOutput.write(contentOfEntry);
zipOutput.closeArchiveEntry();
````

+ 多个文件加入`Zip`压缩

````java
response.setCharacterEncoding("UTF-8");
response.addHeader("Content-Disposition", "attachment;filename=xxx.zip");
response.setContentType("application/zip");
    
List<content> texts = ...; // 此处导出字符串列表
try (ServletOutputStream outStr = response.getOutputStream();
    ZipArchiveOutputStream zipOutput = new ZipArchiveOutputStream(outStr)) {
    for (int i = 0; i < texts.size(); i++) {
        byte[] content = texts.get(i).getBytes(StandardCharsets.UTF_8)
        String entryName = "xxx_" + i + ".txt" 
        ZipArchiveEntry entry = new ZipArchiveEntry (entryName);
        entry.setSize(content.length);
        zipOutput.putArchiveEntry(entry);
        zipOutput.write(content);
        zipOutput.closeArchiveEntry();
    }
} catch (IOException e) {
     e.printStackTrace();
}
````

## 3 示例代码

+ [README](https://github.com/flankx/flankdemo)

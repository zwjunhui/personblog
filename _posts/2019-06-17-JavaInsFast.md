---
layout: post
title: SpringBoot2.0集成FastDFS
date: 2019-06-17
tag: SpringBoot
---

### SpringBoot2.0集成FastDFS

**FastDFS 架构包括 Tracker server 和 Storage server。 客户端请求 Tracker server 进行文件上传、下载，通过 Tracker server 调度最终由 Storage server 完成文件上传和下载。**

**Tracker server 作用是负载均衡和调度，通过 Tracker server 在文件上传时可以根据一些策略找到 Storage server 提供文件上传服务。可以将 tracker 称为追踪服务器或调度服务器。** 

**Storage server 作用是文件存储，客户端上传的文件最终存储在 Storage 服务器上，Storageserver 没有实现自己的文件系统而是利用操作系统 的文件系统来管理文件。可以将storage称为存储服务器。**
![图片](/images/posts/fastdfs/856154-20171012180511480-692747720.png)

#### FastdfsClient配置
```java
package com.example.client;

import com.github.tobato.fastdfs.domain.conn.FdfsWebServer;
import com.github.tobato.fastdfs.domain.fdfs.MetaData;
import com.github.tobato.fastdfs.domain.fdfs.StorePath;
import com.github.tobato.fastdfs.domain.proto.storage.DownloadByteArray;
import com.github.tobato.fastdfs.exception.FdfsUnsupportStorePathException;
import com.github.tobato.fastdfs.service.FastFileStorageClient;
import org.apache.commons.io.FilenameUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.nio.charset.Charset;
import java.util.HashSet;
import java.util.Set;

@Component
public class FastdfsClient {
    private final Logger logger = LoggerFactory.getLogger(FastdfsClient.class);

    @Autowired
    private FastFileStorageClient storageClient;

    @Autowired
    private FdfsWebServer fdfsWebServer;

    /*
     * 前端上传文件
     */
    public String uploadFile(MultipartFile file) throws Exception {
        Set<MetaData> mydata = new HashSet<>();
        StorePath storePath = storageClient.uploadFile(file.getInputStream(), file.getSize(), FilenameUtils.getExtension(file.getOriginalFilename()), mydata);
        logger.info("storePath:{}",JSON.toJSONString(storePath));
        return getResAccessUrl(storePath);
    }

    /*
     * 后端上传文件
     */
    public String uploadFile(File file) throws IOException {
        FileInputStream inputStream = new FileInputStream(file);
        StorePath storePath = storageClient.uploadFile(inputStream, file.length(),
                FilenameUtils.getExtension(file.getName()), null);
        return getResAccessUrl(storePath);
    }

    public String uploadFile(String content, String fileExtension) {
        byte[] buff = content.getBytes(Charset.forName("UTF-8"));
        ByteArrayInputStream stream = new ByteArrayInputStream(buff);
        StorePath storePath = storageClient.uploadFile(stream, buff.length, fileExtension, null);
        return getResAccessUrl(storePath);
    }

    /*
     * 生成图片的完整URl路径
     */
    private String getResAccessUrl(StorePath storePath) {
        String fileUrl = fdfsWebServer.getWebServerUrl() + storePath.getFullPath();
        return fileUrl;
    }

    public byte[] download(String fileUrl) {
        String group = fileUrl.substring(0, fileUrl.indexOf("/"));
        String path = fileUrl.substring(fileUrl.indexOf("/") + 1);
        byte[] bytes = storageClient.downloadFile(group, path, new DownloadByteArray());
        return bytes;
    }

    public void deleteFile(String fileUrl) {
        if (StringUtils.isEmpty(fileUrl)) {
            return;
        }
        try {
            StorePath storePath = StorePath.parseFromUrl(fileUrl);
            storageClient.deleteFile(storePath.getGroup(), storePath.getPath());
        } catch (FdfsUnsupportStorePathException e) {
            logger.warn(e.getMessage());
        }
    }
}

```
#### Controller配置

```java
package com.example.controller;

import com.example.client.FastdfsClient;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Map;

@Controller
@Api(description = "fastdfs接口api")
@RequestMapping("/fdfs")
public class FdfsApi {
    @Autowired
    private FastdfsClient fastdfsClient;

    @RequestMapping("/uploadPage")
    public String uploadPage() {
        return "/index";
    }

    @ApiOperation("文件上传")
    @RequestMapping("/upload")
    @ResponseBody
    public Map<String, Object> upload(MultipartFile file) throws Exception {
        String url = fastdfsClient.uploadFile(file);
        System.out.println(url);
        Map<String, Object> result = new HashMap<>();
        result.put("code", 200);
        result.put("message", "文件上传成功");
        result.put("url", url);
        return result;
    }

    @ApiOperation("文件下载")
    @RequestMapping("/download")
    public void download(String fileUrl, HttpServletResponse response) throws IOException {
        byte[] download = fastdfsClient.download(fileUrl);
        //直接从fileUrl提取出文件名（包含后缀）
        String name = extractName(fileUrl);
        response.setCharacterEncoding("UTF-8");
        response.setHeader("Content-disposition", "attachment;filename=" + URLEncoder.encode(name, "UTF-8"));

        // 写出
        ServletOutputStream outputStream = response.getOutputStream();
        IOUtils.write(download, outputStream);

    }


    @ApiOperation("删除文件")
    @RequestMapping("deleteFile")
    public Map<String, Object> deleteFile(String fileUrl) {
        Map<String, Object> result = new HashMap<>();

        try {
            fastdfsClient.deleteFile(fileUrl);
            result.put("code", 200);
            result.put("msg", "删除成功");
        } catch (Exception e) {
            result.put("code", 201);
            result.put("msg", "删除失败");
        }
        return result;
    }

    /**
     * 从fileUrl中提取出文件名及后缀
     *
     * @param fileUrl
     * @return
     */
    public String extractName(String fileUrl) {
        String name = fileUrl.substring(fileUrl.lastIndexOf("_") + 1);
        if (StringUtils.isBlank(name)) {
            name = fileUrl.substring(fileUrl.lastIndexOf("/") + 1);
        }
        return name;
    }
}


```
#### 配置文件的配置
```javascript
spring:
  servlet:
    multipart:
      max-request-size: 184300000
      max-file-size: 184300000

fdfs:
  so-timeout: 1501
  connect-timeout: 601
  thumb-image: # 缩图
    width: 150
    height: 150
  web-server-url: 192.168.117.130/ 
  tracker-list: # tracker地址 可以配置多个
    - 192.168.117.130:22122 #伺服器ip
```
### 一、依赖

---

```xml
<dependency>
  <groupId>com.jcraft</groupId>
  <artifactId>jsch</artifactId>
  <version>0.1.55</version>
</dependency>
```



### 二、SFTP工具类

---

包含连接服务器、文件上传、文件夹操作等方法。

```java
import com.jcraft.jsch.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;
import java.util.Properties;

/**
 * @Description: SftpUtils
 * @Author: wuchenyang
 * @Date: 2023-5-24 9:27
 * @Version 1.0
 */
public class SftpUtils {

  private static final Logger logger = LoggerFactory.getLogger(SftpUtils.class);

  private SftpUtils() {
  }

  private static Session sshSession = null;

  /**
   * 获取ChannelSftp
   */
  public static ChannelSftp getConnectIP(String host, String sOnlineSftpPort, String username, String password) {
    int port = 0;
    if (!("".equals(sOnlineSftpPort)) && null != sOnlineSftpPort) {
      port = Integer.parseInt(sOnlineSftpPort);
    }
    ChannelSftp sftp = null;
    try {
      JSch jsch = new JSch();
      jsch.getSession(username, host, port);
      sshSession = jsch.getSession(username, host, port);
      sshSession.setPassword(password);
      Properties sshConfig = new Properties();
      sshConfig.put("StrictHostKeyChecking", "no");
      sshSession.setConfig(sshConfig);
      sshSession.connect();
      Channel channel = sshSession.openChannel("sftp");
      channel.connect();
      sftp = (ChannelSftp) channel;
    } catch (Exception e) {
      logger.error("get sftp channel error:", e);
    }
    return sftp;
  }

  /**
   * 上传csv文件
   *
   * @param directory   目标文件夹
   * @param fileName    文件名
   * @param inputStream 输入流
   * @param sftp        sftp通道
   * @return            是否成功
   */
  public static boolean upload(String directory, String fileName, InputStream inputStream, ChannelSftp sftp) {
    try {
      sftp.cd(directory);
      sftp.put(inputStream, fileName);
      return true;
    } catch (Exception e) {
      logger.error("upload to sftp error:", e);
      return false;
    } finally {
      if (sftp.isConnected()) {
        sshSession.disconnect();
        sftp.disconnect();
      }
    }
  }

  public static boolean isExistDir(String path, ChannelSftp sftp) {
    boolean isExist = false;
    try {
      SftpATTRS sftpATTRS = sftp.lstat(path);
      isExist = true;
      return sftpATTRS.isDir();
    } catch (Exception e) {
      if (e.getMessage().equalsIgnoreCase("no such file")) {
        isExist = false;
      }
    }
    return isExist;

  }

  /**
   * 下载
   */
  public static void download(String directory, String downloadFile, String saveFile, ChannelSftp sftp) {
    try {
      sftp.cd(directory);
      File file = new File(saveFile);
      sftp.get(downloadFile, new FileOutputStream(file));
    } catch (Exception e) {
      logger.error("sftp download error:", e);
    } finally {
      if (sftp.isConnected()) {
        sshSession.disconnect();
        sftp.disconnect();
      }

    }
  }

  /**
   * 查看
   */
  public static List<String> check(String directory, ChannelSftp sftp) {
    List<String> fileList = new ArrayList<>();
    try {
      sftp.cd(directory);
      ListIterator a = sftp.ls(directory).listIterator();
      while (a.hasNext()) {
        ChannelSftp.LsEntry oj = (ChannelSftp.LsEntry) a.next();
        logger.info(oj.getFilename());
      }
    } catch (Exception e) {
      logger.error("sftp check error:", e);
    } finally {
      if (sftp.isConnected()) {
        sshSession.disconnect();
        sftp.disconnect();
      }
    }
    return fileList;
  }
}
```
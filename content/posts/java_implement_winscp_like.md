---
title: "Java实现类似WINSCP访问远程Linux服务器，执行命令、上传文件、下载文件"
date: 2020-02-20T10:32:24+08:00
draft: false
categories: ["技术文章"]
---
pom.xml添加的依赖：
```xml
<!-- https://mvnrepository.com/artifact/ch.ethz.ganymed/ganymed-ssh2 --> 
<dependency> 
    <groupId>ch.ethz.ganymed</groupId> 
    <artifactId>ganymed-ssh2</artifactId> 
    <version>262</version> 
</dependency>
```
这里注意，不同版本的这玩意用法有差别。这里我使用的是262。

操作类代码如下：
```java
import ch.ethz.ssh2.*;

import java.io.*;

/**
 * @Author: 风萧古道的博客 windypath.com
 * @Date: 2019/11/29 15:14
 */
public class SSHUtil {

    // uploadFile to Linux

    /**
     * 上传文件到Linux
     * 
     * @param ip ip地址
     * @param username 登录用户名
     * @param password 密码
     * @param remoteFilePath 目标文件所在完整目录
     * @param file 本地文件File对象
     * @return
     */
    public static boolean uploadFile(String ip, String username, String password, String remoteFilePath, File file) {

        FileInputStream input = null;
        BufferedOutputStream boutput = null;
        Connection conn = null;
        try {

            conn = new Connection(ip);
            conn.connect();
            boolean isAuthenticated = conn.authenticateWithPassword(username, password);
            if (isAuthenticated == false) {
                throw new IOException("Authentication failed.");
            }
            SCPClient scpClient = conn.createSCPClient();
            boutput = scpClient.put(file.getName(), file.length(), remoteFilePath, "0600");
            byte[] buffer = new byte[1024 * 8];
            input = new FileInputStream(file);


            int length = -1;
            while ((length = input.read(buffer)) != -1) {
                boutput.write(buffer, 0, length);
            }
            boutput.flush();

        } catch (IOException e) {
            e.printStackTrace(System.err);
        } finally {
            try {

                boutput.close();
                input.close();
                conn.close();
            } catch (IOException e) {
                e.printStackTrace(System.err);
            }
        }
        return true;
    }


  
    /**
     * 从远程服务器上下载文件
     *
     * @param ip ip地址
     * @param username 用户名
     * @param password 密码
     * @param remoteFile 要下载的文件的完整路径
     * @param localFileName 下载到本地的文件名
     * @return
     */
    public static boolean downloadFile(String ip, String username, String password, String remoteFile, String localFileName) {
        SCPInputStream binput = null;
        FileOutputStream output = null;
        BufferedOutputStream boutput = null;
        Connection conn = null;
        try {
            File file = new File(localFileName);
            conn = new Connection(ip);
            conn.connect();
            boolean isAuthenticated = conn.authenticateWithPassword(username, password);
            if (isAuthenticated == false) {
                throw new IOException("Authentication failed.");
            }
            SCPClient scpClient = conn.createSCPClient();
            binput = scpClient.get(remoteFile);
            output = new FileOutputStream(file);
            boutput = new BufferedOutputStream(output);
            byte[] buffer = new byte[1024 * 8];
            int length = -1;
            while ((length = binput.read(buffer)) != -1) {
                boutput.write(buffer, 0, length);
            }


        } catch (IOException e) {
            e.printStackTrace(System.err);
        } finally {
            try {
                binput.close();
                boutput.close();
                output.close();
                conn.close();
            } catch (IOException e) {
                e.printStackTrace(System.err);
            }
        }
        return true;
    }

    /**
     * 获取远程文件的输入流
     *
     * @param ip ip
     * @param username 用户名
     * @param password 密码
     * @param remoteFile 远程文件的完整地址
     * @return 输入流
     */
    public static SCPInputStream getRemoteFileInputStream(String ip, String username, String password, String remoteFile) {
        SCPInputStream binput = null;

        Connection conn = null;
        try {

            conn = new Connection(ip);
            conn.connect();
            boolean isAuthenticated = conn.authenticateWithPassword(username, password);
            if (isAuthenticated == false) {
                throw new IOException("Authentication failed.");
            }
            SCPClient scpClient = conn.createSCPClient();
            binput = scpClient.get(remoteFile);

        } catch (IOException e) {
            e.printStackTrace(System.err);
        }
        return binput;
    }

    /**
     * 执行远程服务器的控制台命令
     *
     * @param ip ip地址
     * @param username 用户名
     * @param password 密码
     * @param cmd 控制台命令
     */
    public static void executeCmd(String ip, String username, String password, String cmd) {
        Connection conn = null;
        try {

            conn = new Connection(ip);
            conn.connect();
            boolean isAuthenticated = conn.authenticateWithPassword(username, password);
            if (isAuthenticated == false) {
                throw new IOException("Authentication failed.");
            }
            Session sess = conn.openSession();
            sess.execCommand(cmd + "\n");
            InputStream stdout = new StreamGobbler(sess.getStdout());
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(stdout));
            while (true) {
                String line = br.readLine();
                if (line == null) {
                    break;
                }
                System.out.println(line);
            }


        } catch (IOException e) {
            e.printStackTrace(System.err);
        } finally {

            conn.close();

        }
    }

}
```

值得一提的是，uploadFile()方法的最后一个参数传入的是指向本地文件的一个File，也就是要上传的文件，而在downloadFile()方法中，最后一个参数是字符串，就是将目标文件下载下来之后，要叫什么名字，记得带上后缀。

执行命令的话，执行一次就新开一次session。一个session（好像貌似）不能做两件事（如上传文件的过程中创建一个新文件夹，再以新文件夹的目录为目标上传目录）。
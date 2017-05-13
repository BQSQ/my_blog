---
title: Oracle文件接口
date: 2017-05-13 14:27:10
tags: [oracle,文件接口]
categories: oracle
keywords: oracle,文件接口,编译java代码
description: 用oracle数据库形式实现文件形式接口
---

在不同的系统进行数据对接中，比较常用的实现方式就是webservices或者http，如果其中一方系统没有集成webservices，做项目过程中立即在原有框架加入该支持，花费的时间和成本相对来说比较高，现在介绍另一种对接形式，文件接口，同样也可以实现不同系统中的数据交互。
<!--more -->
* 优点：   
实现方式简单，在oracle中实现，直接修改存储过程编译，不用重启web服务更新功能，系统不用系统集成webservices框架。   
* 缺点：
不能实时接收数据和反馈。  
--- 
## 需求背景
HR系统将人员信息同步到某系统（暂时叫它X系统），该公司系统之间并没有实现单点登录。需求中人员信息同步不需要实时，同步时间为T+1天。 

## 接口设计

1. HR系统生成文件内容格式：   
工号|姓名|身份证号码|性别编码|性别|出生日期|职级|年龄|移动电话|电子邮件（工作）|入司日期|办公电话|所在系列编码|所在系列名称|当前状态编码|当前状态名称|部门代码|部门名称|加入当前部门时间
2. HR存放人员数据内容的FTP地址：/ftp/data/hr/emp   
3. HR人员数据内容的文件名称格式：EMP\_日期.txt（例如：HR\_EMP_20170101.txt）
4. ftp的用户名和密码，只开放X系统的读取权限。

## 接口实现   

HR系统已经将当天的员工信息按照格式要求生成相应的文件放到ftp中/ftp/data/hr/emp目录下，X系统要做的就是把文件下载下来，然后进行读取数据保存在数据库中。   
### 下载ftp目录的文件（工具类）
oracle可以编译java写的程序，这里我们使用java上传或者下载远程ftp的文件(HR使用的sftp的协议，具体的代码网上很多)。
``` java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.UnsupportedEncodingException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.Properties;
import com.jcraft.jsch.Channel;
import com.jcraft.jsch.ChannelSftp;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.JSchException;
import com.jcraft.jsch.Session;
/**
 * sftp工具类
 */
public class SFtpUtil {
	private static Session session = null;
	// 错误信息
	public static String msg = "";
	/**
	 * 初始化连接
	 * @return
	 * @throws JSchException
	 */
	private static Session initSessionConnect(String ftpHost, int ftpPort,
			String ftpUserName, String ftpPassword) {
		try {
			JSch jsch = new JSch();
			session = jsch.getSession(ftpUserName, ftpHost, ftpPort);
			session.setPassword(ftpPassword);
			session.setTimeout(10000);
			Properties config = new Properties();
			config.put("StrictHostKeyChecking", "no");
			session.setConfig(config);
			session.connect();
		} catch (JSchException e) {
			msg = "ftp连接失败" + e.getMessage();
		}
		return session;
	}

	/**
	 * 从ftp中使用sftp方式下载文件
	 * @param ftpHost   ftp地址
	 * @param ftpUserName  ftp用户名
	 * @param ftpPassword  ftp密码
	 * @param localBasePath 文件保存在本地的路径
	 * @param ftpFileNamePath 人员数据文件
	 * @throws JSchException
	 */
	public static String downloadSftpFile(String ftpHost, String ftpUserName,
			String ftpPassword, String localBasePath, String ftpFileNamePath) {
		Session session = initSessionConnect(ftpHost, 10021, ftpUserName,ftpPassword);
		if ("" == msg) {
			Channel channel = null;
			ChannelSftp chSftp = null;
			String localFilePath = null;
			try {
				channel = session.openChannel("sftp");
				channel.connect();
				chSftp = (ChannelSftp) channel;
				File file = new File(getStandardPath(localBasePath));
				if (!file.exists()) {
					file.mkdir();
				}
				// 人员本地路径
				localFilePath = getStandardPath(localBasePath) + getFileName(ftpFileNamePath);
				chSftp.get(ftpFileNamePath, localFilePath);
			} catch (Exception e) {
				msg = "文件下载失败" + e.getMessage();
			} finally {
				if (null != channel && !channel.isClosed()) {
					channel.disconnect();
				}
				if (null != session && session.isConnected()) {
					session.disconnect();
				}
			}
			/**
			 * 由于在进行文件读取的时候，出现错误，经过很多分析和实验，
			 * 初步判断是oracle读取的时候最后一个字符不能是中文，要不然会找不到\n，现在默认使用@，在oracle中再去掉
			 */
			if(msg == ""){
				replaceFile(localFilePath);
			}
			//删除30天以前的文件
			removeFile(localBasePath);
		}
		return msg;
	}

	/**
	 * sftp上传2个文件
	 * @param ftpHost sftp主机地址
	 * @param ftpUserName 用户名
	 * @param ftpPassword 密码
	 * @param sftpBasePath sftp基本路径
	 * @param localFileNamePath1 本地文件全路径名称1
	 * @param localFileNamePath2 本地文件全路径名称2
	 * @return 成功返回""  失败返回失败信息
	 */
	public static String uploadSftpFile(String ftpHost, String ftpUserName,
			String ftpPassword, String sftpBasePath, String localFileNamePath1,
			String localFileNamePath2){
		Session session = initSessionConnect(ftpHost, 10021, ftpUserName,
				ftpPassword);
		if ("" == msg) {
			Channel channel = null;
			ChannelSftp chSftp = null;
			String ftpFilePath1 = null;
			String ftpFilePath2 = null;
			try {
				channel = session.openChannel("sftp");
				channel.connect();
				chSftp = (ChannelSftp) channel;
				ftpFilePath1 = getStandardPath(sftpBasePath)
						+ getFileName(localFileNamePath1);
				ftpFilePath2 = getStandardPath(sftpBasePath)
						+ getFileName(localFileNamePath2);
				
				chSftp.put(localFileNamePath1, ftpFilePath1,ChannelSftp.OVERWRITE);
				chSftp.put(localFileNamePath2, ftpFilePath2,ChannelSftp.OVERWRITE);
			} catch (Exception e) {
				msg = "上传失败:"+e.getMessage();
			}finally{
				if (null != channel && !channel.isClosed()) {
					channel.disconnect();
				}
				if (null != session && session.isConnected()) {
					session.disconnect();
				}
			}
		}
		return msg;
	}
	
	/**
	 * 获取文件名称
	 * @param filePath
	 * @return
	 */
	private static String getFileName(String filePath) {
		int pos = filePath.lastIndexOf("/");
		return filePath.substring(pos + 1);
	}

	/**
	 * 获取标准路径
	 * @param path
	 * @return
	 */
	private static String getStandardPath(String path) {
		if (path.endsWith("/")) {
			return path;
		} else {
			return path + "/";
		}
	}

	private static void replaceFile(String fileName) {
		StringBuffer sb = new StringBuffer();
		InputStreamReader isr = null;
		BufferedReader br = null;
		FileOutputStream fos = null;
		OutputStreamWriter osw = null;
		try {
			FileInputStream fis = new FileInputStream(fileName);
			isr = new InputStreamReader(fis, "UTF-8");
			br = new BufferedReader(isr);
			String line = null;
			while ((line = br.readLine()) != null) {
				sb.append(line);
				sb.append("@\n"); // 补上换行符
			}
			fos = new FileOutputStream(fileName + "_tmp");
			osw = new OutputStreamWriter(fos, "UTF-8");
			osw.write(sb.toString());
			osw.flush();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
			msg = "文件下载失败" + e.getMessage();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
			msg = "文件下载失败" + e.getMessage();
		} catch (IOException e) {
			e.printStackTrace();
			msg = "文件下载失败" + e.getMessage();
		} finally {
			try {
				if (br != null) {br.close();}
				if (isr != null) {isr.close();}
				if (osw != null) {osw.close();}
				if (fos != null) {fos.close();}
			} catch (IOException e) {
				e.printStackTrace();
				msg = "文件下载失败" + e.getMessage();
			}
			File delFile = new File(fileName);
			delFile.delete();
			File old = new File(fileName + "_tmp");
			File n = new File(fileName);
			old.renameTo(n);
		}
	}

	/**
	 * 删除30天以前的文件
	 */
	private static void removeFile(String localPath) {
		// 计算一周前的日期
		Calendar cdmonth = Calendar.getInstance();
		cdmonth.add(Calendar.DATE, -30);
		Date d = cdmonth.getTime();
		String url = getStandardPath(localPath);
		File fileBag = new File(url);
		String[] filesName = fileBag.list();
		for (int i = 0; i < filesName.length; i++) {
			// 获得文件的创建时间
			File file = new File(url + filesName[i]);
			Long time = file.lastModified();
			Calendar cd = Calendar.getInstance();
			cd.setTimeInMillis(time);
			// 文件的最后一次修改的时间
			Date fileDate = cd.getTime();
			boolean flag = fileDate.before(d);
			if (flag) {
				file.delete();
			}
		}
	}
}
```
**说明：** 
* *代码中的上传文件是2个，一个是数据文件一个是空的.ok文件，是为了让别人读取的时候判断是否数据文件上传成功，或者在.ok文件写入校验码(要有目录写入权限，下载的时候要有读取权限)。由于oracle在读取文件中一行末尾是汉字的时候会有问题，所以将每行末尾加上@符号来避免，暂时不知道是什么原因。本人认为是bug。*
* *引入两个jar包：jsch-0.1.54.jar，jzlib1.1.3.jar*


### 在oracle中编译java代码   

由于我们代码中引入两个jar包：jsch-0.1.54.jar，jzlib1.1.3.jar，所以在oracle中也要加载这两个jar。在jar包目录执行命令。
``` sql
loadjava -r -f -o -user 用户名/密码@IP地址:端口号默认1521:实例名 jsch-0.1.54.jar
loadjava -r -f -o -user 用户名/密码@IP地址:端口号默认1521:实例名 jzlib1.1.3.jar
```
给用户赋权限
``` sql
grant connect  to 用户名;
grant javaidpriv to 用户名;
grant javasyspriv to 用户名;
grant javauserpriv to 用户名;
grant resource to 用户名;
```
建立目录权限   
先建立oracle directory
``` sql
create directory HR_DATA_DIR as '/home/oracle/file/hr_data'; --同步数据使用
```
给directory授权
``` sql
GRANT READ, WRITE ON DIRECTORY HR_DATA_DIR  TO 用户名;
```
为数据库授权，授权文件路径一定要指定到文件所归属的上一层目录；   
注意：(1) 该脚本中的IP及文件目录皆为测试使用，生成环境可能需要更多ip及目录的授权，*代表所有，可自己更改. (2) 以sys用户进行授权

``` sql
begin  
	dbms_java.grant_permission('用户名', 'SYS:oracle.aurora.security.JServerPermission', 'Verifier', '' );
	dbms_java.grant_permission('用户名','java.net.SocketPermission','*','*');
	dbms_java.grant_permission('用户名', 'SYS:java.net.SocketPermission', 'oracle服务器IP地址:*', 'listen,resolve');
	dbms_java.grant_permission('用户名', 'SYS:java.net.SocketPermission', 'oracle服务器ip地址:*', 'connect,resolve');
	dbms_java.grant_permission('用户名', 'SYS:java.io.FilePermission', '*', 'READ,WRITE');
	Dbms_Java.Grant_Permission('用户名', 'SYS:java.lang.RuntimePermission', 'writeFileDescriptor', '');
	Dbms_Java.Grant_Permission('用户名', 'SYS:java.lang.RuntimePermission', 'readFileDescriptor', '');
	dbms_java.grant_permission('用户名', 'SYS:java.io.FilePermission', '<<ALL FILES>>', 'execute' );
    --将ftp文件保存在oracle服务器的位置/home/oracle/file/hr_data/需要进行授权,和目录权限HR_DATA_DIR赋权限不一样
	dbms_java.grant_permission('用户名', 'SYS:java.io.FilePermission', '/home/oracle/file/hr_data/*', 'read,write,execute,delete');
end;
/
```
将上面的ftp工具代码头部加上“create or replace and compile java source named ftpkit as”后直接在oracle中执行就可以了，ftpkit就是该工具的名称。
``` sql
create or replace and compile java source named ftpkit as
import java.io.BufferedReader;
import java.io.File;
...
/**
 * sftp工具类
 */
public class SFtpUtil {
	private static Session session = null;
...
```
### 创建工具类body
将java编译成功之后创建ftp工具类的body，方便供存储过程等调用SFTP_UTIL_PKG。
``` sql
create or replace package SFTP_UTIL_PKG is
  -- Purpose : 文件下载
  -- Created : 2016/11/17 15:50:37
  -- p_host : ftp地址
  -- p_hostname : ftp用户名
  -- p_password : ftp密码
  -- p_local_base_path : 文件保存在本地的路径
  -- p_ftp_path : 数据文件路径
  function sftp_download(p_host            varchar2,
                         p_hostname        varchar2,
                         p_password        varchar2,
                         p_local_base_path varchar2,
                         p_ftp_path       varchar2) return varchar2 as
    language java name 'SFtpUtil.downloadSftpFile(java.lang.String,java.lang.String,java.lang.String,java.lang.String,java.lang.String)  return java.lang.String';
  -- Purpose : 文件上传
  -- Created : 2016/11/17 15:50:37
  -- p_host : ftp地址
  -- p_hostname : ftp用户名
  -- p_password : ftp密码
  -- p_sftp_base_path : 文件上传ftp的路径
  -- p_local_path1 : 本地数据文件路径
  -- p_local_path2 : 本地控制文件路径.ok
  function sftp_upload(p_host           varchar2,
                       p_hostname       varchar2,
                       p_password       varchar2,
                       p_sftp_base_path varchar2,
                       p_local_path1    varchar2,
                       p_local_path2    varchar2) return varchar2 as
    language java name 'SFtpUtil.uploadSftpFile(java.lang.String,java.lang.String,java.lang.String,java.lang.String,java.lang.String,java.lang.String)  return java.lang.String';

end SFTP_UTIL_PKG;
/
create or replace package body SFTP_UTIL_PKG is

end SFTP_UTIL_PKG;
/
```
### 将文件下载读取后存到oracle数据库
其中employee_hr表根据HR提供的字段信息和文件内容进行创建。这里不再给出。
``` sql
create or replace package file_hr_oper_pkg is

  -- Created : 2016/11/17 12:35:20
  -- Purpose : X系统接受hr同步的数据


  --文件名字前缀
  c_hr_emp_name constant varchar2(30) := 'HR_EMP_';
  --文件名后缀
  c_hr_suffix_name constant varchar2(8) := '.txt';

  function download_file return boolean ;

  procedure download_hr_data( p_result  out varchar2);

end file_hr_oper_pkg;
/
create or replace package body file_hr_oper_pkg is


  --下载文件函数
  function download_file return boolean is
  
    v_local_path    varchar2(200);
    v_ftp_host      varchar2(100);
    v_ftp_user      varchar2(100);
    v_ftp_pwd       varchar2(100);
    v_remote_path   varchar2(200);
    v_download_msg  varchar2(2000);
    v_emp_file_name varchar2(200);
  
  begin
    --参数信息
   
    v_local_path:= 下载文件本地存放directory;
	v_ftp_host:= ftp主机地址;
	v_ftp_user:= ftp用户名;
	v_ftp_pwd:=ftp密码;
	v_remote_path:= 远程ftp文件目录/ftp/data/hr/emp;

    --拼装ftp文件信息
    if substr(v_remote_path, length(v_remote_path)) != '/' then
      v_emp_file_name := v_remote_path || '/' ||
                         to_char(sysdate - 1, 'YYYYMMDD') || '/' ||
                         c_hr_emp_name || to_char(sysdate - 1, 'YYYYMMDD') ||
                         c_hr_suffix_name;
    else
      v_emp_file_name := v_remote_path || to_char(sysdate - 1, 'YYYYMMDD') || '/' ||
                         c_hr_emp_name || to_char(sysdate - 1, 'YYYYMMDD') ||
                         c_hr_suffix_name;
    end if;
  
    v_download_msg :=  SFTP_UTIL_PKG.sftp_download(p_host            => v_ftp_host,
                                                      p_hostname        => v_ftp_user,
                                                      p_password        => v_ftp_pwd,
                                                      p_local_base_path => v_local_path,
                                                      p_ftp_path       => v_emp_file_name);
  
    if v_download_msg is not null then
      return false;
    end if;
    return true;
  exception
    when others then
      return false;
  end download_file;

  --读取emp文件数据并保存到表中
  function read_hr_emp_file(p_directory varchar2) return boolean is
  
    v_local_hr_emp_name varchar2(200);
    v_fexists           boolean;
    v_file_length       number;
    v_block_size        binary_integer;
    v_handler           utl_file.file_type;
    v_line_buffer       varchar2(32767);
    v_insert_str        varchar2(32767);
  
  begin
  
    --hr人员数据文件操作  
    v_local_hr_emp_name := c_hr_emp_name ||
                           to_char(sysdate - 1, 'YYYYMMDD') ||
                           c_hr_suffix_name;
    --判断该文件是否存在
    utl_file.fgetattr(location    => p_directory,
                      filename    => v_local_hr_emp_name,
                      fexists     => v_fexists,
                      file_length => v_file_length,
                      block_size  => v_block_size);
  
    if v_fexists then
      v_handler := utl_file.fopen(location  => p_directory,
                                  filename  => v_local_hr_emp_name,
                                  open_mode => 'R');
      --删除表格数据
      delete from employee_hr;
      if utl_file.is_open(v_handler) then
        loop
          begin
            utl_file.get_line(v_handler, v_line_buffer);
            --把一行中末尾@去掉
            select substr(v_line_buffer, 1, length(v_line_buffer) - 1)
              into v_line_buffer
              from dual;
            select REPLACE(v_line_buffer,
                           '|',
                           chr(39) || chr(44) || chr(39))
              into v_insert_str
              from dual;
            v_insert_str := chr(39) || v_insert_str || chr(39);
            v_insert_str := 'insert into employee_hr values(' ||
                            v_insert_str || ')';
            --动态sql插入数据
            EXECUTE IMMEDIATE v_insert_str;
          exception
            WHEN no_data_found THEN
              exit;
          end;
        end loop;
        commit;
        utl_file.fclose(v_handler);
        return true;
      end if;
    else
      return false;
    end if;
  
  exception
    when others then
      utl_file.fclose(v_handler);
      rollback;
      return false;
  end read_hr_emp_file;
  
  --下载主过程
  procedure download_hr_data( p_result  out varchar2) is
  
    v_ora_directory varchar2(200); --文件下载路径
    v_result        varchar(10) := 'N';
  
  begin
    --获取传送文件路径地址 directory 
    v_ora_directory := 'HR_DATA_DIR' ; 
    --下载文件    
    if download_file then
      --下载文件成功   
      --取出文件数据保存到数据库表
      if read_hr_emp_file(p_directory => v_ora_directory) then
        v_result := 'OK';
      end if;
    end if;
    p_result := v_result;
  commit;
  exception
    when others then
      p_result := 'N';
  end download_hr_data;

end file_hr_oper_pkg;
/
```
**说明：**   
*file_hr_oper_pkg是主程序，download_file负责将文件下载到oracle服务器，read_hr_emp_file负责读取本地文件然后使用动态sql保存到oracle数据库中。*


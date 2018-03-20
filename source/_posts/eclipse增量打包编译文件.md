---
title: eclipse增量打包编译文件
date: 2018-03-20 23:48:39
tags: [eclipse,增量打包,发版]
categories: [eclipse,发版]
keywords: [eclipse,增量打包,发版]
---

生产环境发版一般都是增量发版，这个时候需要打包修改的增量文件。使用patch将更新的文件记录导出。需要注意的是对于java匿名内部类编译成class后会多出一个ClassName$1.class这样的文件，所以也要进行拷贝打包。

<!-- more -->

导出patch文件

![导出patch文件](http://opvqbxg2k.bkt.clouddn.com/eclipse/pkg/eclipse-pkg-01.png)
![导出patch文件](http://opvqbxg2k.bkt.clouddn.com/eclipse/pkg/eclipse-pkg-02.png)
![导出patch文件](http://opvqbxg2k.bkt.clouddn.com/eclipse/pkg/eclipse-pkg-03.png)

java代码工具类

``` java
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

/**
 * 增量打包工具类
 */
public class IncrementalPackaging {

	public static final String PATCHFILE="D:/patch.txt";//补丁文件,由eclipse svn plugin生成  
    
    public static final String PROJECTPATH="D:/kurrent_dev/CEM_V3.0.0_hq";//项目文件夹路径  
      
    public static final String WEBCONTENT="webapp";//web应用文件夹名
      
    public static final String CLASSPATH="D:/kurrent_dev/CEM_V3.0.0_hq/target/classes";//class存放路径  
      
    public static final String DESPATH="D:/update_pkg";//补丁文件包存放路径  
    
    public static final String VERSION="20180320";//补丁版本 
    
    
    
	public static void main(String[] args) {
		copyFiles(getPatchFileList());
	}
	/**
	 * 得到patch里面更改过的文件
	 * @return
	 * @throws Exception
	 */
	public static List<String> getPatchFileList(){  
		FileInputStream f = null;   
		BufferedReader dr = null;  
		List<String> fileList = new ArrayList<String>(); 
		try {
			f = new FileInputStream(PATCHFILE);  
			dr =new BufferedReader(new InputStreamReader(f,"utf-8"));
        	String line;  
        	while((line=dr.readLine())!=null){   
        		if(line.indexOf("Index:") == 0){  
        			line=line.replaceAll(" ","");  
        			line=line.substring(line.indexOf(":")+1,line.length());  
        			fileList.add(line);
        			System.out.println("文件集合:"+line);
        		}
        	}
        	System.out.println("---------------------------------------------------------------------------------------------");
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("文件集合error");
		}finally{
			if (null != dr) {
				try {
					dr.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if(null != f){
				try {
					f.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		
		//文件处理
		handle$File(fileList);
		
		return fileList;
    } 
	
	/**
	 * copy 文件 ，有一部分文件含有匿名内部类，会生成ClassName$1.class类型的文件，需要用正则表达式获取文件
	 * @param list
	 */
	public static void copyFiles(List<String> list){  
		if(null == list || list.size() <=0 ){
			return ;
		}
		System.out.println("开始copy文件-------------------------------------------------------------------------");
        for(String fullFileName:list){  
            if(fullFileName.indexOf("src/main")!=-1){//对源文件目录下的文件处理 
            	//java文件处理
            	if(fullFileName.indexOf("src/main/java")!= -1){
            		String fileName=fullFileName.replace("src/main/java","");  
            		if(fileName.endsWith(".java")){  
                        fileName=fileName.replace(".java",".class");  
                    }
            		
            		fullFileName=CLASSPATH+fileName;
            		
            		String tempDesPath=fileName.substring(0,fileName.lastIndexOf("/"));
            		
            		String desFilePathStr=DESPATH+"/"+VERSION+"/WEB-INF/classes"+tempDesPath;  
            		File desFilePath=new File(desFilePathStr);  
            		if(!desFilePath.exists()){  
            			desFilePath.mkdirs();  
            		}  
            		String desFileNameStr=DESPATH+"/"+VERSION+"/WEB-INF/classes"+fileName;  
            		
            		copyFile(fullFileName, desFileNameStr);  
            		System.out.println("class文件:" + fullFileName+" 复制完成"); 
            		
            	}else if (fullFileName.indexOf("src/main/resources")!= -1){
            		//资源文件，直接从项目文件中拷贝
            		String fileName=fullFileName.replace("src/main/resources",""); 
            		
            		fullFileName=PROJECTPATH+"/"+fullFileName;//将要复制的文件全路径 
            		
            		String desFileNameStr=DESPATH+"/"+VERSION+"/WEB-INF/classes"+fileName; 
            		
            		String tempDesPathStr=desFileNameStr.substring(0,desFileNameStr.lastIndexOf("/"));
            		
            		File desFilePath=new File(tempDesPathStr);  
            		if(!desFilePath.exists()){  
            			desFilePath.mkdirs();  
            		}
            		copyFile(fullFileName, desFileNameStr);
            		System.out.println("资源文件:"+fullFileName+"复制完成");  
            	}else if(fullFileName.indexOf("src/main/webapp")!= -1){
            		//普通文件处理
            		String fileName=fullFileName.replace("src/main/webapp",""); 
            		
            		fullFileName=PROJECTPATH+"/"+fullFileName;//将要复制的文件全路径 
            		
            		String desFileNameStr=DESPATH+"/"+VERSION+fileName; 
            		
            		String tempDesPathStr=desFileNameStr.substring(0,desFileNameStr.lastIndexOf("/"));
            		
            		File desFilePath=new File(tempDesPathStr);  
            		if(!desFilePath.exists()){  
            			desFilePath.mkdirs();  
            		}
            		copyFile(fullFileName, desFileNameStr);
            		System.out.println("普通文件:"+fullFileName+"复制完成");  
            		
            	}else{
            		System.err.println("其他情况文件未考虑到:"+fullFileName);
            	}
                
            }else{
            	System.err.println("其他情况文件未考虑到:"+fullFileName);
            }  
              
        }  
          
    } 
	
	/**
	 * 处理匿名内部类等的编译.class问题
	 */
	private static void handle$File(List<String> list){
		List<String> list$ = new ArrayList<>();
		for(String fullName : list){
			if(fullName.endsWith(".java")){
				//只有编译java有这种情况
				String fileClassName = CLASSPATH + fullName.replace("src/main/java", "").replace(".java", "");
				String fileName$ = fileClassName.substring(fileClassName.lastIndexOf("/")+1)+"$" ;
				String fileClassPath = fileClassName.substring(0, fileClassName.lastIndexOf("/"));
				File file = new File(fileClassPath);
				//必须是路径
				if (file.isFile() || !file.exists()) {
		            continue;
		        }
				//列出所有文件
				File[] files = file.listFiles();
				//进行className$匹配
				for(File f : files){
					if(f.isDirectory()){
						continue;
					}
					if(f.getName().startsWith(fileName$)){
						String fullFile$ = fullName.substring(0, fullName.lastIndexOf("/")+1)+f.getName();
						System.out.println("带有$情况的文件:"+fullFile$);
						list$.add(fullFile$);
					}
				}
				
			}
		}
		list.addAll(list$);
	}
	
	
 	private static void copyFile(String sourceFileNameStr, String desFileNameStr) {  
        File srcFile=new File(sourceFileNameStr);  
        File desFile=new File(desFileNameStr);  
        try {  
            copyFile(srcFile, desFile);  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
	
	
	public static void copyFile(File sourceFile, File targetFile) throws IOException {  
        BufferedInputStream inBuff = null;  
        BufferedOutputStream outBuff = null;  
        try {  
            // 新建文件输入流并对它进行缓冲  
            inBuff = new BufferedInputStream(new FileInputStream(sourceFile));  
  
            // 新建文件输出流并对它进行缓冲  
            outBuff = new BufferedOutputStream(new FileOutputStream(targetFile));  
  
            // 缓冲数组  
            byte[] b = new byte[1024 * 5];  
            int len;  
            while ((len = inBuff.read(b)) != -1) {  
                outBuff.write(b, 0, len);  
            }  
            // 刷新此缓冲的输出流  
            outBuff.flush();  
        } finally {  
            // 关闭流  
            if (inBuff != null)  
                inBuff.close();  
            if (outBuff != null)  
                outBuff.close();  
        }  
    }  
}
```

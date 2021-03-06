# #Java 读取 .properties 配置文件的几种方式

Java 开发中，需要将一些易变的配置参数放置再 XML 配置文件或者 properties 配置文件中。然而 XML 配置文件需要通过 DOM 或 SAX 方式解析，而读取 properties 配置文件就比较容易。

## #下面是读取.properties文件的几种方式：

### #基于ClassLoder读取配置文件

注意：该方式只能读取类路径下的配置文件，有局限但是如果配置文件在类路径下比较方便。

```code
Properties properties = new Properties();
//使用ClassLoader加载properties配置文件生成对应的输入流
InputStream in = PropertiesMain.class.getClassLoader().getResourceAsStream("config/config.properties");
//使用properties对象加载输入流
properties.load(in);
//获取key对应的value值
properties.getProperty(String key);
```

### #基于 InputStream 读取配置文件

注意：该方式的优点在于可以读取任意路径下的配置文件

```code
Properties properties = new Properties();
// 使用InPutStream流读取properties文件
BufferedReader bufferedReader = new BufferedReader(new FileReader("E:/config.properties"));
properties.load(bufferedReader);
// 获取key对应的value值
properties.getProperty(String key);
```

### #通过 java.util.ResourceBundle 类来读取，这种方式比使用 Properties 要方便一些

1. 通过 ResourceBundle.getBundle() 静态方法来获取（ResourceBundle是一个抽象类），这种方式来获取properties属性文件不需要加.properties后缀名，只需要文件名即可

```code
//config为属性文件名，放在包com.test.config下，如果是放在src下，直接用config即可  
ResourceBundle resource = ResourceBundle.getBundle("com/test/config/config");
String key = resource.getString("keyWord"); 
```

2. 从 InputStream 中读取，获取 InputStream 的方法和上面一样，不再赘述

```code
ResourceBundle resource = new PropertyResourceBundle(inStream);
```

***注意：***
在使用中遇到的最大的问题可能是配置文件的路径问题，如果配置文件入在当前类所在的包下，那么需要使用包名限定，如：`config.properties`入在`com.test.config`包下，则要使用`com/test/config/config.properties`（通过Properties来获取）或`com/test/config/config`（通过ResourceBundle来获取）；属性文件在src根目录下，则直接使用`config.properties`或`config`即可。

## #测试代码

```code
package com.test.properties;

import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Enumeration;
import java.util.Properties;

import org.springframework.core.io.support.PropertiesLoaderUtils;

/**
 * 
 * @ClassName: TestProperties   
 * @Description: 获取配置文件信息  
 * @date: 2017年11月25日 上午10:56:00  
 * @version: 1.0.0
 */
public class TestProperties {
    
    
    /**
     * 
     * @Title: printAllProperty   
     * @Description: 输出所有配置信息  
     * @param props
     * @return void  
     * @throws
     */
    private static void printAllProperty(Properties props)  
    {  
        @SuppressWarnings("rawtypes")  
        Enumeration en = props.propertyNames();  
        while (en.hasMoreElements())  
        {  
            String key = (String) en.nextElement();  
            String value = props.getProperty(key);  
            System.out.println(key + " : " + value);  
        }  
    }

    /**
     * 根据key读取value
     * 
     * @Title: getProperties_1   
     * @Description: 第一种方式：根据文件名使用spring中的工具类进行解析  
     *                  filePath是相对路劲，文件需在classpath目录下
     *                   比如：config.properties在包com.test.config下， 
     *                路径就是com/test/config/config.properties    
     *          
     * @param filePath 
     * @param keyWord      
     * @return
     * @return String  
     * @throws
     */
    public static String getProperties_1(String filePath, String keyWord){
        Properties prop = null;
        String value = null;
        try {
            // 通过Spring中的PropertiesLoaderUtils工具类进行获取
            prop = PropertiesLoaderUtils.loadAllProperties(filePath);
            // 根据关键字查询相应的值
            value = prop.getProperty(keyWord);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return value;
    }
    
    /**
     * 读取配置文件所有信息
     * 
     * @Title: getProperties_1   
     * @Description: 第一种方式：根据文件名使用Spring中的工具类进行解析  
     *                  filePath是相对路劲，文件需在classpath目录下
     *                   比如：config.properties在包com.test.config下， 
     *                路径就是com/test/config/config.properties
     *              
     * @param filePath 
     * @return void  
     * @throws
     */
    public static void getProperties_1(String filePath){
        Properties prop = null;
        try {
            // 通过Spring中的PropertiesLoaderUtils工具类进行获取
            prop = PropertiesLoaderUtils.loadAllProperties(filePath);
            printAllProperty(prop);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 根据key读取value
     * 
     * @Title: getProperties_2   
     * @Description: 第二种方式：使用缓冲输入流读取配置文件，然后将其加载，再按需操作
     *                    绝对路径或相对路径， 如果是相对路径，则从当前项目下的目录开始计算， 
     *                  如：当前项目路径/config/config.properties, 
     *                  相对路径就是config/config.properties   
     *           
     * @param filePath
     * @param keyWord
     * @return
     * @return String  
     * @throws
     */
    public static String getProperties_2(String filePath, String keyWord){
        Properties prop = new Properties();
        String value = null;
        try {
            // 通过输入缓冲流进行读取配置文件
            InputStream InputStream = new BufferedInputStream(new FileInputStream(new File(filePath)));
            // 加载输入流
            prop.load(InputStream);
            // 根据关键字获取value值
            value = prop.getProperty(keyWord);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return value;
    }
    
    /**
     * 读取配置文件所有信息
     * 
     * @Title: getProperties_2   
     * @Description: 第二种方式：使用缓冲输入流读取配置文件，然后将其加载，再按需操作
     *                    绝对路径或相对路径， 如果是相对路径，则从当前项目下的目录开始计算， 
     *                  如：当前项目路径/config/config.properties, 
     *                  相对路径就是config/config.properties   
     *           
     * @param filePath
     * @return void
     * @throws
     */
    public static void getProperties_2(String filePath){
        Properties prop = new Properties();
        try {
            // 通过输入缓冲流进行读取配置文件
            InputStream InputStream = new BufferedInputStream(new FileInputStream(new File(filePath)));
            // 加载输入流
            prop.load(InputStream);
            printAllProperty(prop);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 根据key读取value
     * 
     * @Title: getProperties_3   
     * @Description: 第三种方式：
     *                    相对路径， properties文件需在classpath目录下， 
     *                  比如：config.properties在包com.test.config下， 
     *                  路径就是/com/test/config/config.properties 
     * @param filePath
     * @param keyWord
     * @return
     * @return String  
     * @throws
     */
    public static String getProperties_3(String filePath, String keyWord){
        Properties prop = new Properties();
        String value = null;
        try {
            InputStream inputStream = TestProperties.class.getResourceAsStream(filePath);
            prop.load(inputStream);
            value = prop.getProperty(keyWord);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return value;
    }
    
    /**
     * 读取配置文件所有信息
     * 
     * @Title: getProperties_3   
     * @Description: 第三种方式：
     *                    相对路径， properties文件需在classpath目录下， 
     *                  比如：config.properties在包com.test.config下， 
     *                  路径就是/com/test/config/config.properties 
     * @param filePath
     * @return
     * @throws
     */
    public static void getProperties_3(String filePath){
        Properties prop = new Properties();
        try {
            InputStream inputStream = TestProperties.class.getResourceAsStream(filePath);
            prop.load(inputStream);
            printAllProperty(prop);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    
    public static void main(String[] args) {
        // 注意路径问题
        String properties_1 = getProperties_1("com/test/config/config.properties", "wechat_appid");
        System.out.println("wechat_appid = " + properties_1);
        getProperties_1("com/test/config/config.properties");
        System.out.println("*********************************************");
        // 注意路径问题
        String properties_2 = getProperties_2("configure/configure.properties", "jdbc.url");
        System.out.println("jdbc.url = " + properties_2);
        getProperties_2("configure/configure.properties");
        System.out.println("*********************************************");
        // 注意路径问题
        String properties_3 = getProperties_3("/com/test/config/config.properties", "wechat_appid");
        System.out.println("wechat_appid = " + properties_3);
        getProperties_3("/com/test/config/config.properties");
    }
}
```

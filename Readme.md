Java通过JNA与Golang交互

示例
   
   java 调用端 https://github.com/voyage-1969/java-jna-golang-example 
   golang实现端 https://github.com/voyage-1969/golang-jna-example

什么是JNA
   
    JNA(Java Native Access)是一个开源的Java框架，是Sun公司推出的一种调用本地方法的技术，是建立在经典的JNI基础之上的一个框架。
    之所以说它是JNI的替 代者，是因为JNA大大简化了调用本地方法的过程，使用很方便，基本上不需要脱离Java环境就可以完成。

为什么用JNA
    
    跨语言调用有很多方式JNA的好处就是直接调用速度快，比起RPC更效率
    打包处理方便，通过grpc也可以进行java与golang的通信，但不推荐


JNA 基本组成
    
    1.接口 
        java语言需要以native方法 接口的方式在java程序中调用
    2.动态链接库
        win平台一般是.dll，linux平台一般是.so 是一段以C/C++编写的程序
    3.类型映射
        java基本类型与go语言基本类型之间映射
        
Go语言中cgo使用方法
    
    使用JNA需要借助cgo与c语言交互
    引用的C头文件需要在注释中声明，紧接着注释需要有import "C"，且这一行和注释之间不能有空格
    声明使用cgo
    package main
    //引入Cgo
    import "C"
    
    //export FunctionTest
    func FunctionTest(pub string) *C.char {
    	return C.CString(pub)
    }
    如上 声明一个JNA方法
    这里使用string作为参数的话，需要处理字符串映射
    返回值是C语言String类型
    
构建动态链接库dll/so
    
    win环境
    go build -buildmode=c-shared -o 文件名.dll .\go文件名.go
    linux环境
    go build -buildmode=c-shared -o 文件名.so .\go文件名.go
    
Java中调用程序
    
    Maven中添加依赖
    
    <dependency>
         <groupId>net.java.dev.jna</groupId>
         <artifactId>jna</artifactId>
         <version>5.6.0</version>
    </dependency>
    
    接口编写
    
    public interface Lib extends Library {
        /**
         * CGO 字符串类型特殊处理 go语言字符串类型映射
         */
        public class GoString extends Structure {
            public static class ByValue extends GoString implements Structure.ByValue {
            }
    
            public String p;
            public long n;
    
            protected List getFieldOrder() {
                return Arrays.asList(new String[]{"p", "n"});
            }
        }
    
        /**
         * JNA 方法
         *
         * @param pub
         * @return
         */
        String FunctionTest(GoString.ByValue pub);
    }
    
    引入动态链接库
    
    public class TestUtils {
        public static Lib LIB;
        private static final String WIN_DLL = "动态链接库名.dll";
        static {
            //注意这里可能有路径问题 即找不到DLL
            LIB = Native.load(WIN_DLL, Lib.class);
        }
    
        //调用示例
        public static void main(String[] args) {
            String str = LIB.FunctionTest(toGoString("Hello"));
            System.out.println(str);
        }
        
        //类型转换
        private static Lib.GoString.ByValue toGoString(String str) {
            Lib.GoString.ByValue str3 = new Lib.GoString.ByValue();
            str3.p = str;
            str3.n = str3.p.length();
            return str3;
        }
    }
    
找不到DLL/so文件 路径问题解决
    
    win10环境和Linux环境 是默认不含Dll路径的，可以通过以下方法引入DLL/SO文件
    //环境DLL加载
    NativeLibrary.addSearchPath(WIN_DLL, rootPath);

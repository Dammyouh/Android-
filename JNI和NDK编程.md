- 通过JNI，用户可以调用C，C++所编写的本地代码
- NDK：是Android所提供的一个工具集合，通过NDK可以在Android中更加方便的通过JNI来访问本地代码，除此之外，NDK还提供了交叉编译器，开发人员只需要更改mk文件，就可以生成CPU平台的动态库。
- JNI的数据类型：基本类型（jboolean,jbyte,jchar,jshort,jint,jlong,jfloat,jdouble,void)和引用类型(类，对象，数组）
---
title: AS的GsonFormat插件变种
date: 2018-01-16 14:00:58
tags: code
---



Bean的生成会用到一些插件来完成，比如[GsonFormat](https://github.com/zzz40500/GsonFormat)，但是后台会有情况比如生成的一个类A，但是后台给这个类A的值不是一个空对象，而是null。这样导致我们在调用类A的cc属性的时候，因为A已经是null了还调用null的一个属性而导致空指针异常，所以这个插件为了就是解决这个问题，自动补充null的时候的值，所有代码都是改别人的库，变种而已。更改的代码如下：

```
     String method = "public "
                            .concat(typeStr).concat(
                                    "   get").concat(
                                    captureName(field)).concat(
                                    "() { " +getNotNull(typeStr, field1.getGenerateFieldName())+"  return ").concat(
                                    field1.getGenerateFieldName()).concat(" ;} ");
                    mClass.add(mFactory.createMethodFromText(method, mClass));
```



在这个原来生成get的方法代码中插入一个getNotNull字符，方法内容如下:

```java
private String getNotNull(String typeStr,String fieldName) {
        String result="";
        if (typeStr.equals("String")){//
            result="if("+fieldName+"==null){\n"+fieldName+"=\"\";}";
            return result;
        }
        if (typeStr.equals("int")
                |typeStr.equals("double")
                |typeStr.equals("long")
                |typeStr.equals("float")
                |typeStr.equals("byte")
                |typeStr.equals("short")
                |typeStr.equals("char")){
            return result;
        }else{
            String prx=typeStr;
            if (typeStr.startsWith("java.util.List")){
                prx=prx.replace("java.util.List","java.util.ArrayList");
            }
            result="if("+fieldName+"==null){\n"+fieldName+"= new "+prx+"();}";
        }
        return result;
    }

```



如果不是基本数据类型，而且是null值，就new一个出来，如果还有报错，可以以后再修正。就是一个静态代码的生成罢了

代码地址https://github.com/yeqinfu/NotNullGsonFormat












---
title: OpenGL入门
date: 2018-08-14 14:00:58
tags: code
---



总结一下demo的知识点

```kotlin
 override fun onSurfaceCreated(glUnused: GL10?, config: EGLConfig?) {
        glClearColor(0.0f, 0.0f, 0.0f, 0.0f)
        table = Table()
        mallet = Mallet()
        textureProgram = TextureShaderProgram(context, R.raw.texture_vertex_shader, R.raw.texture_fragment_shader)
        colorProgram = ColorShaderPorgram(context, R.raw.simple_vertex_shader, R.raw.simple_fragment_shader)
        texture = TextureHelper.loadTexture(context, R.drawable.icon_welcome)

    }
//略过
```

```kotlin
override fun onSurfaceChanged(p0: GL10?, width: Int, height: Int) {
        glViewport(0, 0, width, height)
        Logger.d("==============" + projectionMatrix.contentToString())
        MatrixHepler.perspectiveM(projectionMatrix, 45f, width.toFloat() / height, 1f, 10f)
        Logger.d("==============" + projectionMatrix.contentToString())
        //设置一个单位矩阵
        Matrix.setIdentityM(modelMatrix, 0)
        //单位矩阵z平移-2
        Matrix.translateM(modelMatrix, 0, 0f, 0f, -2.5f)


        var temp = FloatArray(16)
        //矩阵相乘
        Matrix.multiplyMM(temp, 0, projectionMatrix, 0, modelMatrix, 0)
        System.arraycopy(temp, 0, projectionMatrix, 0, temp.size)

        //Matrix.translateM(projectionMatrix, 0, 0f, 0f, -2.5f)
        Matrix.rotateM(projectionMatrix, 0, -60f, 1f, 0f, 0f)

    }
```

主要看onSurfaceChanged方法，投射矩阵，平移矩阵，旋转矩阵

### MatrixHepler.perspectiveM 参数传入了表示45度投射

![image](http://ws3.sinaimg.cn/large/c1b251b3gy1fu98lsnntkj21500mf477.jpg)

```kotlin
fun perspectiveM(m:FloatArray,yFovInDegrees:Float,aspect:Float,n:Float,f:Float){
        /**计算焦距*/
        //角度转弧度
        val angleInRadians=(yFovInDegrees*Math.PI/180.0).toFloat()
        //焦距
        val a=(1.0/Math.tan(angleInRadians/2.0)).toFloat()
        /**投影矩阵*/
        m[0]=a/aspect
        m[1]=0f
        m[2]=0f
        m[3]=0f
        //=========
        m[4]=0f
        m[5]=a
        m[6]=0f
        m[7]=0f
        //=========
        m[8]=0f
        m[9]=0f
        m[10]=-((f+n)/(f-n))
        m[11]=-1f
        m[12]=0f
        m[13]=0f
        m[14]=-((2f*f*n)/(f-n))
        m[15]=0f

    }
```



如果把onSurfaceChanged最后一行的旋转先去掉，从摄像机的**顶部**看，投影矩阵各个字段的意思

![image](http://wx1.sinaimg.cn/large/c1b251b3gy1fu99l989hdj20k60idwfg.jpg)

当前n=1而f=10这个比例展示出的图像如下



![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fu99m533ngj20hi0rfjsx.jpg)

明显视野两端还有缝隙，那么现在计算出一个比例，刚好让它填满屏幕来证明，那个草图没错。






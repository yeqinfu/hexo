---
title: androidannotations generate plugin
date: 2016-11-21 13:31:56
tags: plugin
---

项目中使用androidannotations，但是androidstudio没有这个的生成插件。自己简单写了个，先看效果：

 ![b](/imgs/b.gif)



github地址：https://github.com/yeqinfu/Plugin_QW_JSON_FORMAT

用idea写的。

* 首先用swing写一个弹窗界面，用来输入输出。没东西，拖拖控件就出来了。

* 接着写一个action，这个action代表android studio的一个菜单功能或者一个快捷键

* ```java
  public class Action_AndroidAnnotations extends AnAction
  ```

  这个就是主要类，继承 AnAction 

* 实现一个方法：

  ```java
  public void actionPerformed(AnActionEvent e) {
      UI_AndroidAnnotationsFormat ui_androidAnnotationsFormat=new UI_AndroidAnnotationsFormat();
      ui_androidAnnotationsFormat.showDialog();
      mEditor = e.getData(PlatformDataKeys.EDITOR);
      mProject = e.getData(PlatformDataKeys.PROJECT);
      ui_androidAnnotationsFormat.setInterfaceInsert(new UI_AndroidAnnotationsFormat.formatInterface(){

          @Override
          public void insertResult(String insertStr) {
              insertSomething( insertStr);
          }
      });

  }
  ```

  这个方法在快捷键被按下的时候激活，这边弹出刚做的窗口就行了。showdialog后面的代码没什么卵用，😄不会被调用。我懒的删除。

  ```java
  public void initUIAction() {
      frame = new JFrame("xml 转android annotations");
      frame.setContentPane(jpanel);
      frame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
      frame.pack();
      Dimension screensize = Toolkit.getDefaultToolkit().getScreenSize();
      int Swing1x = 500;
      int Swing1y = 300;
      frame.setBounds(screensize.width / 2 - Swing1x / 2, screensize.height / 2 - Swing1y / 2, Swing1x, Swing1y);
      button1.addActionListener(new ActionListener() {
          @Override
          public void actionPerformed(ActionEvent e) {
              makeFile();
          }
  ```


      });
  }
  ```

  这个方法是初始化界面；重点是makefile方法，对字符串进行了转换。

  ```java
  private void makeFile() {

      String str = textArea1.getText();
      Document document = Utils_XmlPraser.xmlToDocument(str);
      if (document!=null){
          // 获取根节点元素对象
          Element root = document.getRootElement();
          String result = Utils_XmlPraser.elementFindId(root);
          textArea1.setText(result);
      }else{
          Messages.showMessageDialog("爷,你xml格式错误啦,我解释不了", "Information", Messages.getInformationIcon());
      }
  }
  ```

  这个方法中

```java
  String result = Utils_XmlPraser.elementFindId(root);
```

  这个是重点。

```java
  public static String elementFindId(Element node){
      String result="";
      // 使用递归
      Iterator<Element> iterator = node.elementIterator();
      while (iterator.hasNext()) { // 遍历里面的节点
          Element e = iterator.next();
          // 首先获取当前节点的所有属性节点
          List<Attribute> list = e.attributes();
          // 遍历节点属性
          for (Attribute attr : list) {
              if (attr.getName().equals("id")){//如果是id节点
                  result+=" \n@ViewById\n"+e.getName()+"  "+attr.getValue().substring(attr.getValue().indexOf("/")+1)+";";
                  break;
              }
          }
              result+=elementFindId(e);
      }

      return result;
  }
```

  递归每个xml节点然后把它拼起来。

  完了

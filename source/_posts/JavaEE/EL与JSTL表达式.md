---
title: EL与JSTL表达式
date: 2018-01-30 14:15
categories:
  - JavaEE
tags:
  - JavaEE
  - JSP
toc: true
---

#### EL 表达式

EL 表达式的功能:(让 JSP 编写更为简单)

1. EL 是 Expression Language 的简称，即表达式语言；
2. EL 在 JSP 中使用，服务器会对其进行解析翻译，生成相应的 Java 代码；
3. EL 的作用是用来在 JSP 页面输出动态内容，可以替代 JSP 中的表达式元素<%=%>

**EL 表达式的一般格式:**
`${EL表达式}`
例如:`${param.username}`
等同于:`<%=request.getParameter("username")%>`

#### EL 表达式的内置对象

其中

- 2 个内置对象为了方便输出请求参数： param/paramValues； + 内置对象 param：用来输出请求参数的值，格式为\${param.请求参数名字} + 内置对象 paramValues：用来获取一对多的参数值，返回一个数组。
- 4 个内置对象为了方便输出各个范围的属性： pageScope/ requestScope /sessionScope /applicationScope + 获取四个范围的属性数据 + 检索顺序：当不指定范围时，例如，\${user.pwd}，将自动从 pageScope 开始查找，直到 applicationScope，如果没查到，则什么也不显示
- 2 个与请求头有关的内置对象：header/headerValues + 内置对象 header：用来输出输出某一个请求头的值，格式为\${header.请求头名字} + 内置对象 headerValues：如果某个请求头的值有多个，则使用 headerValues 返回一个数组。
- 2 个其他内置对象：cookie/initParam + 内置对象 cookie：用来获取 cookie 的值 + 内置对象 initParam：用来输出上下文参数;
- 1 个特殊的内置对象 pageContext + 内置对象 pageContext：EL 中的 pageContext 对象可以调用 PageContext 类中所有符合规范的 getXxx 方法

#### 使用 EL 取出内置对象的数据

1. 普通对象和对象属性。
   服务器端：
   `request.setAttribute("student", student);`
   在浏览器上打印出服务器端绑定的数据：

   ```html
   ${ student }
   <!-- 相当于执行了 student.toString(); -->
   ${ student.name }
   <!-- 相当于执行了 student.getName(); -->
   ${ student.teacher.name }
   <!-- 相当于执行了 student.getTeacher().getName(); -->
   ```

2. 数组中的数据。
   服务器端：

   ```java
    String[] nameArray = new String[]{"Tom", "Lucy", "Lilei"};
    request.setAttribute(“nameArray”,nameArray);
    Student[] students = new Student[3];
    students[0] = stu1;
    students[1] = stu2;
    students[2] = stu3;
    request.setAttribute(“students”,students);
   ```

   在浏览器上打印出服务器端绑定的数组数据：

   ```html
   ${ nameArray[0] }
   <!-- Tom -->
   ${ nameArray[1] }
   <!-- Lucy -->
   ${ nameArray[2] }
   <!-- Lilei -->
   ${ students[0].id }
   <!-- 输出第一个学生的ID -->
   ${ students[1].name }
   <!-- 输出第二个学生的name -->
   ${ students[2].teacher.name }
   <!-- 输出第三个学生的老师的name -->
   ```

3. List 中的数据。
   服务器端：

   ```java
    List<Student> studentList=new ArrayList<Student>();
    studentList.add(stu1);
    studentList.add(stu2);
    studentList.add(stu3);
    request.setAttribute(“studentList”,studentList);
   ```

   在浏览器上打印出服务器端绑定的 List 数据：

   ```html
   ${ studentList[0].id }
   <!-- 输出第一个学生的ID -->
   ${ studentList[1].name }
   <!-- 输出第二个学生的name -->
   ${ studentList[2].teacher.name }
   <!-- 输出第三个学生的老师的name -->
   ```

4. Map 中的数据。
   服务器端：

   ```java
    Map<String, Student> studentMap = new HashMap<String, Student>();
    studentMap.put("Tom", stu1);
    studentMap.put("Lucy", stu2);
    studentMap.put("Lilei", stu3);
    request.setAttribute(“studentMap”,studentMap);
   ```

   在浏览器上打印出服务器端绑定的 Map 数据：

   ```html
   ${ studentMap.Tom.id }
   <!-- 输出第一个学生的ID -->
   ${ studentMap.Lucy.name }
   <!-- 输出第二个学生的name -->
   ${ studentMap.Lilei.teacher.name }
   <!-- 输出第三个学生的老师的name -->
   ```

#### EL 运算符

EL 中提供了多种运算符，可以对变量或常量进行运算，输出运算结果；
EL 中的运算符包括：

1. 算术运算符
2. 比较运算符
3. 逻辑运算符
4. 其他运算符

#### JSTL

JSTL 是一套定义好的标签库，可以直接使用；
JSTL 的全称是 Jsp Standard Tag Library，即 JSP 标准标签库；
JSTL 包含很多标签，根据其作用可以分为：属性相关的标签、条件分支相关的标签、迭代标签、其他标签；
标签库包括标签处理器类及描述文件 tld 文件，JSTL 也一样：

- 使用 JSTL 首先需要下载相关的 jar 文件并保存到工程的 lib 目录下；在 JSP 中使用 taglib 指令引入需要使用的标签库；
- forEach、set、if 等是 JSTL 中常用的标签；
- JSTL 标签库的使用是为类弥补 html 表的不足，规范自定义标签的使用而诞生的。在告别 modle1 模式开发应用程序后，人们开始注重软件的分层设计，不希望在 jsp 页面中出现 java 逻辑代码，同时也由于自定义标签的开发难度较大和不利于技术标准化产生了自定义标签库。

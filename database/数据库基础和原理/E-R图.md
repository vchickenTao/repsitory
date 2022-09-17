**<span style="font-size: 35px;">🔏 E-R图</span>**

---

Entity-Relationship，有三个组成部分: 实体、属性、联系。

用来进行关系型数据库系统的概念设计。

### 实体的三种联系实体的三种联系

包含一对一，一对多，多对多三种。

- 如果 A 到 B 是一对多关系，那么画个带箭头的线段指向 B；
- 如果是一对一，画两个带箭头的线段；
- 如果是多对多，画两个不带箭头的线段。

下图的 Course 和 Student 是一对多的关系。

![实体的三种联系实体的三种联系](https://www.pdai.tech/_images/pics/292b4a35-4507-4256-84ff-c218f108ee31.jpg)

### 表示出现多次的关系

一个实体在联系出现几次，就要用几条线连接。

下图表示一个课程的先修关系，先修关系出现两个 Course 实体，第一个是先修课程，后一个是后修课程，因此需要用两条线来表示这种关系。

![表示出现多次的关系](https://www.pdai.tech/_images/pics/8b798007-e0fb-420c-b981-ead215692417.jpg)

### 联系的多向性

虽然老师可以开设多门课，并且可以教授多名学生，但是对于特定的学生和课程，只有一个老师教授，这就构成了一个三元联系。

![联系的多向性](https://www.pdai.tech/_images/pics/423f2a40-bee1-488e-b460-8e76c48ee560.png)

一般只使用二元联系，可以把多元联系转换为二元联系。

![image](https://www.pdai.tech/_images/pics/de9b9ea0-1327-4865-93e5-6f805c48bc9e.png)

### 表示子类

用一个三角形和两条线来连接类和子类，与子类有关的属性和联系都连到子类上，而与父类和子类都有关的连到父类上。

![表示子类](https://www.pdai.tech/_images/pics/7ec9d619-fa60-4a2b-95aa-bf1a62aad408.jpg)



### 参考文章

- [Java 全栈知识体系](https://www.pdai.tech/md/db/sql/sql-db-theory.html#%E6%A6%82%E5%BF%B5)
- [CS-Notes](http://www.cyc2018.xyz/%E6%95%B0%E6%8D%AE%E5%BA%93/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86.html)


# Spring Date JPA

## 多表操作

### 一对一

1、**一方维护关系，另一方放弃维护关系   一对一     一般让关联关系少的一方维护关系**

2、**一对一中 外键要加唯一约束** @JoinColumn(unique=true)

3、**cascade 级联 在操作谁就在谁里配置级联**



```java
#文章表article					   内容表article_date     
aid    #主键						id         #主键				
author #作者						content    #内容   			
title  #标题						article_id #外键			

//文章
public class Article{
    private int aid;
    private String author;
    private String title;
    
     //@OneToOne 使用注解声明表间关系
     //mappedBy="article" 声明主动放弃维护关系
     //cascade=CascadeType.PERSIST  级联保存 在保存Article时，同时保存ArticleDate 
    @OneToOne(mappedBy="article",cascade=CascadeType.PERSIST)  
    private ArticleDate articleDate;//声明类间关系 一篇文章对应一段内容
}
//文章内容    
public class ArticleDate{
    private int id;
    private String content;
   
    @OneToOne   //使用注解声明表间关系
    @JoinColumn(name="article_id",referencedColumnName="aid",unique=true) //声明维护关系
    private Article article;//声明类间关系 一段内容对应一篇文章
}
/**
1、声明类间关系
2、使用注解声明表间关系
3、明确谁来维护关系  一方维护关系 另一方放弃维护关系  一对一中 一般让关联关系少的一方维护
@JoinColumn
	unique 唯一约束 unique=true表示外键唯一 是一对一  unique=false是一对多
	name 当前表的外键  referencedColumnName 对应另一张表的主键
@OneToOne
	mappedBy="当前类在对方类中的属性名"    声明主动放弃维护关系   
	cascade 级联 在操作谁就在谁里配置级联 如保存Article时级联保存ArticleDate 级联配置在Article
*/
```



### 一对多

1、一方维护关系，另一方放弃维护关系   **一对多   一般让多（Many的一方）的一方维护关系**



```java
#文章表article					       评论表comment
aid     #主键							cid     #主键
author  #作者							comment #评论
title   #标题							aid     #外键

//文章
public class Article{
    private int aid;
    private String author;
    private String title;
    
    @OneToMany(mappedBy="article")
    private List<Comment> comments=new ArrayList<>();//类间关系 一篇文章有多个评论
     
}
//文章评论    
public class Comment{
    private int cid;
    private String comment;
   
    @ManyToOne
    @JoinColumn(name="aid",referencedColumnName="aid",unique=false)//一对多 关系维护在Many方
    private Article article;//类间关系 多个评论属于一篇文章
}
/**
1、声明类间关系
2、使用注解声明表间关系
3、明确谁来维护关系  一方维护关系 另一方放弃维护关系 一对多 一般让多(Many方)的一方维护
@JoinColumn
	unique 唯一约束   unique=false 是一对多 unique默认false
	name 当前表的外键  referencedColumnName 对应另一张表的主键
@OneToMany
	mappedBy="当前类在对方类中的属性名"    声明主动放弃维护关系   
*/
```





### 多对多

1、一方维护关系，另一方放弃维护关系   **多对多   一般让关联关系少的一方维护关系**



```java
#文章表article			#文章类型表article_typpe		       文章类型表type
aid     #主键			 tid    #文章类型id 				 tid     #主键
author  #作者			 aid    #文章id				      name    #类型名称
title   #标题							

//文章
public class Article{
    private int aid;
    private String author;
    private String title;
    
   @ManyToMany(mappedBy="articles")//放弃关系维护
   private List<String> types;//类间关系 一篇文章可以有多个类型
     
}
//文章类型    
public class Type{
    private int tid;
    private String name;
   
    @ManyToMany
    @JoinTable(
        //name 中间表名称
    	name="article_type",
    	//中间表的外键对应到当前表的主键 name外键 referencedColumnName主键
    	joinColumns={@JoinColumn(name="tid",referencedColumnName="tid")},
    	//中间表的外键对应到对方表的主键 name外键 referencedColumnName主键
    	inverseJoinColumns={@JionColumn(name="aid",referencedColumnName="aid")}
    )
    private List<String> articles;//类间关系 一个类型可以属于多个文章
    
}
/**
1、声明类间关系
2、使用注解声明表间关系
3、明确谁来维护关系  一方维护关系 另一方放弃维护关系 多对多对多 一般让关联关系少的一方维护
@JoinTable
	name 当前表的外键  referencedColumnName 对应另一张表的主键
@ManyToMany
	mappedBy="当前类在对方类中的属性名"    声明主动放弃维护关系   
*/
```














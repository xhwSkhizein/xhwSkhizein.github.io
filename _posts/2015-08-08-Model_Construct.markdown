---
title: Model_Construct
layout: post
guid: urn:uuid:ad2a755f-99e3-4c4d-ab9e-275fff799677
tags:
  - java
  - Design Patterns
---
### 问题：
    如何构建一个可以适应需求变化的model？

> 问题描述：
  在实际生产环境中model是根据需求随时可变的，因此产生了一些可选属性，在构造具有可选属性的model时怎样构建才最合理呢？

	1) 假如有一个model，
    	public class User {
			private final String name;
			private final int age;            //optional
			private final String address;     //optional
		}
	大多数属性都是可选的，在构建这个对象时，就会出现多个适配不同参数的构造器, 当model属性多的时候简直是场灾难😅。
	其实有一种好的方式来解决这个问题，那就是通过model内部的构造者来构造model，大概是这个样子：
        public class TestModelConstruct {
			private final String name;
			private final String address;
			private final int age;
			public TestModelConstruct(Builder builder) {
				this.name = builder.name;
				this.age = builder.age;
				this.address = builder.address;
			}
			public String getName() {
				return name;
			}
			public String getAddress() {
				return address;
			}
			public int getAge() {
				return age;
			}
			public static class Builder {
				private final String name;
				private String address;
				private int age;
				public Builder(String name) {
					super();
					this.name = name;
				}
				public Builder age(int age) {
					this.age = age;
					return this;
				}
				public Builder address(String address) {
					this.address = address;
					return this;
				}
				public TestModelConstruct build() {
					return new TestModelConstruct(this);
				}
			}
		}

		使用方法：
			new TestModelConstruct.Builder("Atom") //
				.age(21) //
				.address("Beijing China") //
				.build();

		缺点：
			a. 当我们有多个这样的model时，就会发现我们一直在重复写builder，代码实在是太丑了，So DRY！
			b. model一般映射成数据库的表，如果可选属性也被映射成字段，那表里岂不都是null，如果不映射成字段那可选属性怎么存储？

		IDEA_I：
		通过map来存储可选属性，即将可选属性集存储成一个map，key对应属性名，value对应属性值，然后将map压缩后存储到数据库，
		取出时在转化成map，这样当model新增可选属性后，就是向map中添加一个键值对就好了。
		public class TestModelConstruct {
			private final String name;
			private Map<String, Object> attrMap = new HashMap<>();
			public TestModelConstruct(Builder builder) {
				this.name = builder.name;
				this.attrMap = builder.attrMap;
			}
			public String getName() {
				return name;
			}
			public String getAddress() {
				return (String) this.attrMap.get("address");
			}
			public int getAge() {
				return (Integer) this.attrMap.get("age");
			}
			public static class Builder {
				private final String name;
				private Map<String, Object> attrMap = new HashMap<>();
				public Builder(String name) {
					super();
					this.name = name;
				}
				public Builder age(int age) {
					this.attrMap.put("age", age);
					return this;
				}
				public Builder address(String address) {
					this.attrMap.put("address", address);
					return this;
				}
				public TestModelConstruct build() {
					return new TestModelConstruct(this);
				}
			}
		然而并没有解决之前重复写builder的问题，但是通过map我们将可选属性从model中隐匿了，无论再怎么删减
		model属性，model的数据结构是不会那么轻易改变了，它的表应该就是这个样子：👇
				create table model(
					id bigint unsigned not null auto_increment,
					name varchar(128) charset utf8mb4 collate utf8mb4_general_cii not null,
					attrs varchar(2048) not null,  -- (一个字段搞定n个可选属性)
					status tinyint unsigned not null,
					create_time datetime not null,
					primary key(id),
					unique key uniq_name(name))
				Engine=InnoDB ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=4; --(打开压缩)

		重新审视缺点：
			a. 可选字段虽然隐匿了，但看见性对使用者也降低了，取可选属性时的key存在极大的问题，值也不是强类型的
			b. DRY
		解决思路：
			I. 抽象出一个接口来定义可选属性，设置统一的key，通过key可以反射出指定类型的value，
		然后通过简单的get(K<V> k)方法获取特定key的value。


直接上代码: [_click_me_to_see_code_](http://github.com/xhwSkhizein/atom/tree/master/src/main/java/im/atom/base/framework/entity/attr)

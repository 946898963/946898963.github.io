---
title: Builder模式
date: 2016-06-17 20:06:52
categories:
- 技术
- 设计模式
tags:
- 设计模式
---



## 介绍

Builder模式是一步一步的创建一个复杂对象的创建型模式，它允许用户在不知道内部的构建细节的情况下，可以更加精细的控制对象的构造流程。该模式是为了将构建复杂对象的过程和他的部件解耦，使得构建过程和部件的表示隔离开。

因为一个复杂对象有很多大量组成部分，如汽车，有车轮，方向盘，发动机，还有各种小零件等，如何将这些部件装配成一辆汽车，这个装配过程很漫长，也很复杂，对于这种情况，为了在构建过程中对外部隐藏实现细节，就可以使用Builder模式将部件跟组装过程分离，使得构建过程和部件都可以自由扩展，两者之间的耦合降到最低。

将一个复杂对象的构建跟他的表示分离，使得同样的构建过程可以创建不同的表示。

## 使用场景

> * 相同的方法，不同的执行顺序，产生不同的结果。
> * 多个部件或者零件都可以装配到一个对象中，但是产生的运行结果又不同。
> * 产品类非常复杂，或者产品类中的调用顺序不同产生了不同的作用，这个时候使用建造者模式非常合适。
> * 当初始化一个对象非常复杂，如参数多，且很多参数都有默认值的时候。

## 角色介绍

> * Product产品类----产品的抽象类
> * Builder----抽象Builder类，规范产品组建，一般是由子类实现具体的组建过程 
> * ConcreteBuilder----具体的Builder类
> * Director---统一组装过程

## 简单实现

计算机的组装过程比较复杂，而且顺序是不固定的，为了易于理解，我们把计算机的组装过程简化为，构建主机，设置操作系统，设置显示器三个部分，然后通过Director和具体的Builder来构建计算机对象。

抽象的计算机类
```
public abstract class Computer {

    protected String mBoard;
    protected String mDisplay;
    protected String mOs;

    protected Computer(){

    }

    public void setBoard(String board){
        this.mBoard= board;
    }

    public void setDisplay(String display){
        this.mDisplay=display;
    }

    public abstract void setOs();

    @Override
    public String toString() {
        return "mBoard:"+mBoard+"mDisplay:"+mDisplay+"mOs:"+mOs;
    }
}

```
具体的计算机类
```
public class MacComputer extends Computer {

    protected MacComputer(){

    }


    @Override
    public void setOs() {
        mOs="mac os";
    }
}

```
抽象的Builder
```
public abstract class Builder {

    public abstract void buildBoard(String board);
    public abstract void buildDisplay(String display);
    public abstract void buildOs();
    public abstract Computer create();

}

```
具体的Builder
```
public class MacBuilder extends Builder {
    private Computer computer=new MacComputer();
    @Override
    public void buildBoard(String board) {
        computer.setBoard(board);
    }

    @Override
    public void buildDisplay(String display) {
        computer.setDisplay(display);
    }

    @Override
    public void buildOs() {
        computer.setOs();
    }

    @Override
    public Computer create() {
        return computer;
    }
}
```
Director类
```
public class Director {

    private Builder builder;

    public Director(Builder builder){
        this.builder=builder;
    }

    public void construct(String board,String display){
        builder.buildBoard(board);
        builder.buildDisplay(display);
        builder.buildOs();
    }

}
```

```
       Builder builder = new MacBuilder();
       Director director = new Director(builder);
       director.construct("主板", "显示器");
       System.out.println(builder.create().toString());
```

通过具体的MacBuilder构建MacBook对象，而Director封装了构建复杂对象的过程，对外隐藏构建细节。Builder跟Director一起将一个复杂对象的构建跟表示分离，使得同样的构建过程可以创建不同的对象。

在现实的开发中Director经常被省略。而直接使用一个Builder类来进行对象的组装，这个Builder通常为链式调用，他的关键点是每个setter方法都将返回自身，这样就使得setter方法可以链式调用，代码大致如下：
new Builder().setA("A").setB("B").setC("C");通过这种形式，不仅去除了Director角色。整个结构也更加简单，也能对Product对象的组装过程有更加精细的控制。

省略Director的情况

```
public abstract class BuilderB {

    public abstract BuilderB buildBoard(String board);
    public abstract BuilderB buildDisplay(String display);
    public abstract BuilderB buildOs();
    public abstract Computer create();

}

```


```
public class MacBuilderB extends BuilderB {

    private Computer computer=new MacComputer();
    
    @Override
    public BuilderB buildBoard(String board) {
        computer.setBoard(board);
        return this;
    }

    @Override
    public BuilderB buildDisplay(String display) {
        computer.setDisplay(display);
        return this;
    }

    @Override
    public BuilderB buildOs() {
        computer.setOs();
        return this;
    }

    @Override
    public Computer create() {
        return computer;
    }
}

```

```
public static void main(String[] args) {
    BuilderB builderB = new MacBuilderB();
	System.out.println(builderB.buildBoard("主板").buildDisplay("显示器").buildOs().create());

}
```
---
title: 游戏AI初探
date: 2023-08-30 14:34:54
categories:
  - 游戏相关
tags: GameDev
---



# randomness

## 高斯分布

用于模拟生活中的一些行为, 可以是物理特性，认知能力，消遣时间或子弹分布。

+ Average or max speed 
+ Average or max acceleration 
+ Size, width, height, or mass
+ Visual or physical reaction time 
+ Fire or reload rate for firing 
+ Refresh rate or cool-down rate for healing or special abilities 
+ Chance of missing or striking a critical hit

## 随机数过滤

为了留住玩家，可以通过对随机数进行过滤，来让玩家或对手获得优势

## 噪声

最后，柏林噪声不仅仅是图像领域，1D柏林噪声可以用于平滑过渡动作，动画，和许多其他的行为。

+ Movement (direction, speed, acceleration) 
+ Layered onto animation (adding noise to facial movement or gaze [Perlin 97]) 
+ Accuracy (winning or losing streaks, being in the groove, luck, or success) 
+ Attention (guard alertness, response time) • Play style (defensive, offensive) 
+ Mood (calm, angry, happy, sad, depressed, manic, bored, engaged)

# FSM

```java
public abstract class FSMTransition {
    // 判断当前转换是否成立，如果为真，有条路从当前状态走到目标状态
    abstract boolean isValid();
	// 得到状态转换的最终状态
    abstract FSMachine getNextState();
	// 发生在原状态和目标状态中间的事
    abstract void onTransition();
}
```



```java
public abstract static class FSMachine {
    // 进入状态
    abstract void onEnter();
    // 每tick更新
    abstract void onUpdate();
    // 离开状态
    abstract void onExit();
    // 可选的状态转换路线
    List<FSMTransition> transitions = new ArrayList<>();
}
```



```java
public static class FiniteStateMachine {
    // 主要更新方法
    void update(){
        // 判断当前状态的所有可能转换路线
        for (FSMTransition transition : activeState.transitions) {
            if (transition.isValid()) { // 如果可以选择该路线
                activeState.onExit(); 
                activeState = transition.getNextState();
                transition.onTransition();
                activeState.onEnter();
                break;
            }
        }
        // 更新状态
        activeState.onUpdate();
    };
    // 状态机所持有的所有状态
    List<FSMachine> states;
    // 初始状态
    FSMachine initState;
    // 当前状态
    FSMachine activeState;
}
```

需要注意的是，具体的逻辑在`onExit(), onUpdate(), onExit(), onTransition()`这四个方法中实现，其中，`onTransition()`方法应与具体AI逻辑无关，如日志记录，全局事件等.

## Hierarchical FSM

当开发到晚期时，状态会几十上百个，增加新状态时会很复杂且容易出错。

> 在HFSM中，每个单独的状态都是一个状态机。可以使整个状态机分离到单独的多个状态机中.

## Behavior Trees

a example:

```java
public abstract static class Behavior {
    abstract void onInit(); // 初次进入
    abstract State update(); // 更新状态
    abstract void onTerminate(State); // 离开
}
```

行为树的实现可以通过`装饰器`或`组合`模式去实现。

新的行为树可以多个实体共享，使用DSL，用于多种逻辑.

## Utility Systems

效用系统，用于处理非布尔值信息。比起饿了吃饭，效用系统计算了多饿，多重等非01信息。通过多种因素计算出一个分数，或者加上一个随机值，根据最终的值进行选择。

## Goal-Oriented Action Planners

目标导向型行为选择

>  GOAP is derived from the Stanford Research Institute Problem Solver 

给予一个目标，然后在所有可选的方法中自行选择，让AI自动选择。
























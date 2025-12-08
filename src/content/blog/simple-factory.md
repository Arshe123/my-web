---
title: 设计模式-简单工厂模式
description: '简单工厂设计模式'
publishDate: 2025-12-08 20:35:59
tags:
  - design-pattern
  - simple-factory
---

## 前言

简单工厂设计模式是工厂模式中最简单的一种，以下用计算器的例子来介绍。

通常来说让你写一个简易计算器代码，也许你写出的代码如下（假设所有输入都是合法的）：

```java
public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        while (true) {
            // 第一步：输入第一个数字
            double num1 = 0;
            System.out.print("请输入第一个数字或输入exit退出：");
            String input1 = scanner.nextLine().trim();
            
            // 输入exit退出
            if ("exit".equalsIgnoreCase(input1)) {
                System.out.println("计算器已退出，感谢使用！");
                break;
            }

            num1 = Double.parseDouble(input1);

            // 第二步：输入操作符
            String operator = "";
            System.out.print("请输入运算符（+、-、*、/）：");
            String operator = scanner.nextLine().trim();

            // 第三步：输入第二个数字
            double num2 = 0;
            System.out.print("请输入第二个数字：");
            String input2 = scanner.nextLine().trim();
            num2 = Double.parseDouble(input2);

            double result = calculate(num1, num2, operator);
            System.out.printf("\n✅ 计算结果：%.2f %s %.2f = %.2f\n\n", num1, operator, num2, result);
        }
        scanner.close();
    }

    /**
     * 计算方法
     * @param num1
     * @param num2
     * @param operator
     * @return
     */
    private static double calculate(double num1, double num2, String operator) {
        return switch (operator) {
            case "+" -> num1 + num2;
            case "-" -> num1 - num2;
            case "*" -> num1 * num2;
            case "/" -> {
                if (num2 == 0) {
                    throw new ArithmeticException("除数不能为0");
                }
                yield num1 / num2;
            }
        };
    }
}
```

这样确实实现了计算器的功能，但是它并不利于维护。

如果我要加新一种运算，比如求余数，那么我需要修改`Calculator`类中的`calculate`方法，新加一种`case`去判断。
这并不符合面向对象的思想。

考虑使用简单工厂模式来实现。

## 简单工厂实现

UML图：

![url](https://cos.arshe.cn/simple-factory/url.png)

代码如下：

```java
/**
 * 运算抽象类
 */
public abstract class Operation {
  public abstract double calculate(double num1, double num2);
}

/**
 * 加法运算
 */
public class Add extends Operation {
  @Override
  public double calculate(double num1, double num2) {
    return num1 + num2;
  }
}

/**
 * 减法运算
 */
public class Sub extends Operation {
  @Override
  public double calculate(double num1, double num2) {
    return num1 - num2;
  }
}

/**
 * 乘法运算
 */
public class Mul extends Operation {
  @Override
  public double calculate(double num1, double num2) {
    return num1 * num2;
  }
}

/**
 * 除法运算
 */
public class Div extends Operation {
  @Override
  public double calculate(double num1, double num2) {
    if (num2 == 0) throw new RuntimeException("除数不能为0");
    return num1 / num2;
  }
}

/**
 * 工厂类
 */
public class OperationFactory {
  public static Operation createOperation(String operate) {
    switch (operate) {
      case "+":
        return new Add();
      case "-":
        return new Sub();
      case "*":
        return new Mul();
      case "/":
        return new Div();
      default:
        return null;
    }
  }
}
```

客户端代码如下：

```java
// 获取运算类
Operation operation = OperationFactory.createOperation(operate);
// 计算结果
result = operation.calculate(num1, num2);
```

这样便可从客服端解耦，只需要靠工厂类获取计算类对象来运算即可获取结果。

当我需要添加新的运算符时，只需要在工厂类中添加新的运算符即可，不需要修改客户端代码。
比如添加一个指数运算：

```java
/**
 * 指数运算
 */
public class Pow extends Operation { // [!code ++]
  @Override // [!code ++]
  public double calculate(double num1, double num2) { // [!code ++]
    return Math.pow(num1, num2); // [!code ++]
  } // [!code ++]
} // [!code ++]

/**
 * 修改后的工厂类
 */
public class OperationFactory {
  public static Operation createOperation(String operate) {
    switch (operate) {
      case "+":
        return new Add();
      case "-":
        return new Sub();
      case "*":
        return new Mul();
      case "/":
        return new Div();
      case "pow": // [!code ++]
        return new Pow(); // [!code ++]
      default:
        return null;
    }
  }
}
```

虽然客服端不再做逻辑判断，拓展性了好了很多。但是也存在缺陷，每次增加新的运算，也需要去修改工厂类的代码。后续介绍优化的工厂模式。

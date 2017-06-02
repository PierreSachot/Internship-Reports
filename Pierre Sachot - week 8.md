# Week 8

## Context

This week, with Yannick Mayeur, we needed to increase the test code coverage of Dataset classes inside of Kichwa coders' January library. To do that, we used a plugin named "EclEmma" which allows you to see the coverage percentage in each classes and when you open a class to see which lines are executed by your tests and which ones are not.

It looks like this:

![example coverage](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%208/Screenshot_1.png?raw=true)

## Useful tests

Test coverage is a really usefull feature, which allows you to see the percentage of your class which is tested, and to know where you have possible bugs, because a non tested code is a buggy code, that's a fondamental rule! You need to test your code to be sure that everything is working correctly.

For example in this function, some cases are not tested, this is a problem:

![non tested parts](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%208/Screenshot_2.png?raw=true)

Even if you do not want to test 100% of your code because it would be to long and ask you too many time, so try to test the most used cases and cases which are critic. 

Coverage tests allow you to understand how your code is checked and the global coverage of your entire project, it allows you to have a global view:

![global view](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%208/Screenshot_3.png?raw=true)

Or a more specific view:

![specific view](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%208/Screenshot_4.png?raw=true)

But even if you test the most part of your code, this does not mean that you are testing it well.

## Limits

Even if use code coverage is a really useful indicator, it's not all the time good and does not mean that your tests are right, this is not because or your code are in green and covered that it is a code without bugs...

For example take this code:

```Java
/**
* Return the double of the entered value
* @return double of the entered value
**/
public static int returnDouble(int val){
  return 4;
}
```

if you write this test:

```Java
@Test
public void testReturnDouble(){
  int input = 2;
  int expected = 4;
  
  int output = MyClass.returnDouble(2);
  
  assertEquals(expected, output);
}
```

This test will pass, and the code coverage will be at 100%, all the code will be test, the problem is that the test is completely wrong, because this test is working only if you enter 2.

## Test Driven Development

This is a technic used to be sure that the code is right and allows to simplify the development of a function, because you already know the special cases and the things you need to be careful about.

This is a really simple principal: 
  
  - Think about the special cases of your function
  - Write the tests before coding anything
  - Write your function code, see if your test pass
  
It allows the developer to do real tests on his function, to do not tests only the cases he thinks about and to have a more complete test.

For more informations, you can check this article: [TDD developement](http://agiledata.org/essays/tdd.html)

But do not let it become a tragedy: [The tragedy of 100% code coverage](http://labs.ig.com/code-coverage-100-percent-tragedy)


## Conclusion

I hope you understood the usefulness of using coverage tests on your project, and the limits of it, a program cannot tell you the utility of your test, it can only checks the result and makes stats. Nothing can replace your brain. :)

# Week 6:

## Context:

This week, I was back on January project, more specifically on Junit Tests. We needed to test two function of the Maths.java which are: arctan2() function and abs() to calculate the absolute value of a Dataset.
I worked more on the second function, and as those two functions were really similar so we decided to create a ParameterizeTest class to include the first funcion too. This is a test that could be applied to both series of code, with only variable changing, it can be used to test a function with a lot of values and see which one is failling, or like in our case by changing the class type.

## Tests before using Parameterized Tests:

As you can see bellow, those two tests were really similar:

```Java
@Test
public void testAbsDoubleInput() {
  double[] obj = new double[] { 4.2, -2.9, 6.1 };
  Dataset a = DatasetFactory.createFromObject(obj);

  int size = a.getSize();
  double[] c = new double[size];
  for (int i = 0; i < size; i++) {
    double abs = Math.abs(obj[i]);
    c[i] = abs;
  }
  Dataset expectedResult = DatasetFactory.createFromObject(c);

  Dataset actualResult = Maths.abs(a);
  TestUtils.assertDatasetEquals(expectedResult, actualResult, true, ABSERRD, ABSERRD);
}
 
@Test
public void testAbsbyteInput() {
  Dataset a = DatasetFactory.createFromObject(new byte[] { -4, 8, 6 });

  int size = a.getSize();
  byte[] c = new byte[size];
  for (int i = 0; i < size; i++) {
    int abs = Math.abs(a.getByte(i));
    c[i] = (byte) abs;
  }
  Dataset expectedResult = DatasetFactory.createFromObject(c);

  Dataset actualResult = Maths.abs(a);
  TestUtils.assertDatasetEquals(expectedResult, actualResult, true, ABSERRD, ABSERRD);
}

```

Here I only put the example for 2 classes, but we used it for 6 different classes.So now we needed to identify what was similar and what was different in all the tests, and what could be changed to make them more similar.

Here the `actualResult` needed to be in the result type we wanted, because that what's we needed to test, but the expected result type could be written however we wanted it:

So now you need to identify what is similar and what is changing in all the tests, what you can change to be more similar. Here the actualResult need to be in the result type we want, because that what's we wanted to test, but the expected result type can be as we want:

```Java
public void testAbsDoubleInput() {
  double[] obj = new double[] { 4.2, -2.9, 6.1 };
  Dataset a = DatasetFactory.createFromObject(DoubleDataset.class, obj);

  int size = a.getSize();
  double[] c = new double[size];
  for (int i = 0; i < size; i++) {
    double abs = Math.abs(obj[i]);
    c[i] = abs;
  }
  Dataset expectedResult = DatasetFactory.createFromObject(DoubleDataset.class, c);

  Dataset actualResult = Maths.abs(a);
  TestUtils.assertDatasetEquals(expectedResult, actualResult, true, ABSERRD, ABSERRD);
}

@Test
public void testAbsbyteInput() {
  double[] obj = new double[] { 4.2, -2.9, 6.1 };
  Dataset a = DatasetFactory.createFromObject(ByteDataset.class, obj);

  int size = a.getSize();
  double[] c = new double[size];
  for (int i = 0; i < size; i++) {
    double abs = Math.abs(obj[i]);
    c[i] = abs;
  }
  Dataset expectedResult = DatasetFactory.createFromObject(ByteDataset.class, c);

  Dataset actualResult = Maths.abs(a);
  TestUtils.assertDatasetEquals(expectedResult, actualResult, true, ABSERRD, ABSERRD);
```

Here in the Dataset constructor, you can see that we are created a ByteDataset from a double array. This is possible because Dataset class allows the user to do it. Now you can see that the only thing that the only thing that will change in our tests is the class variable to create the Dataset.

So, you can write a variable which will take the class type, like that:

```Java
@Test
public void testAbsbyteInput() {
    Class<? extends Dataset> class1 = ByteDataset.class;
		double[] obj = new double[] {4.2, -2.9, 6.10};
		Dataset input = DatasetFactory.createFromObject(class1, obj);
    ....
}
```

## Tests using Parameterized Tests:

So once you can write a parameterize class test to reduce your code size and simplify your tests:

```Java
package org.eclipse.january.dataset;

import org.junit.Test;
import org.eclipse.january.asserts.TestUtils;

import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameter;
import org.junit.runners.Parameterized.Parameters;
import org.junit.runner.RunWith;

import java.util.Arrays;
import java.util.Collection;

@RunWith(Parameterized.class)
public class MathsBasicTypeAbsFunctionParameterizeTest {

	@Parameters(name = "{index}: {0}") //the "(name = "{index}: {0}")" allows the Junit test to write which test failed with which parameter
	public static Collection<Object> data() { //called to get the array of variables that need to be change
		return Arrays.asList(new Object[] { FloatDataset.class, DoubleDataset.class, ByteDataset.class,
				ShortDataset.class, IntegerDataset.class, LongDataset.class });
	}

	@Parameter
	public Class<? extends Dataset> classType; //Parameter which change when the last test is done.

  //parameter specific to our test don't worry about this one
	private final static double ABSERRD = 1e-8;

	@Test
	public void test() {
		Class<? extends Dataset> class1 = classType;
		double[] obj = new double[] {4.2, -2.9, 6.10};
		Dataset input = DatasetFactory.createFromObject(class1, obj);
		Dataset output = DatasetFactory.createFromObject(class1, new double[]{0,0,0});

		int size = input.getSize();
		double[] c = new double[size];
		for (int i = 0; i < size; i++) {
			double abs = Math.abs(obj[i]);
			c[i] = abs;
		}
		Dataset expectedResult = DatasetFactory.createFromObject(class1, c);

		Dataset actualResult = Maths.abs(input, output);
		TestUtils.assertDatasetEquals(expectedResult, actualResult, true, ABSERRD, ABSERRD);
	}
}
```

Here, the function data() is the one which will be called to change the data type. Now you have a parameterize test which will work with every class which extends Dataset.

## Conclusion:

Now you know how to reduce your tests code, identify things which are the same between your tests, it's now possible to code tests efficiently, by winning time and avoiding code duplication, this is why Parameterized Tests are really usefull and you need to use them.

![Test](https://github.com/WhiteBlackGoose/GenericTensor/workflows/Test/badge.svg)
![GitHub](https://img.shields.io/github/license/WhiteBlackGoose/GenericTensor?color=blue)

# GenericTensor

It is the only fully-implemented generic-tensor library for C#. Allows to work with tensors of custom types.
Is still under development, as well as documentation.

Tensor - is an extentension of matrices, a N-dimensional array. Soon you will find all common functions that are
defined for matrices and vectors here. In order to make it custom, Tensor class is generic, which means that
you could use not only built-in types like int, float, etc., but also your own types.

### Hello world

For full correct work you will need some methods for your type to be defined. But first, we could
create our first int tensor:

```cs
var myTensor = new GenTensor<int>(3, 4, 5);
```

Alright, your first tensor is created. You can now access its members:
```cs
myTensor[2, 0, 3] = 5;
Console.WriteLine(myTensor[1, 1, 1]);
```

However, if you try to do anything with it,
for example, add it to itself:

```cs
var b = GenTensor<int>.PiecewiseAdd(myTensor, myTensor);
```

it will require you to first define static method Add:

```cs
ConstantsAndFunctions<int>.Add = (a, b) => a + b;
```

There is how BuiltinTypeInitter.InitForInt() is implemented (call it to make Tensor<int> work):

```cs
public static void InitForInt()
{
    ConstantsAndFunctions<int>.Add = (a, b) => a + b;
    ConstantsAndFunctions<int>.Subtract = (a, b) => a - b;
    ConstantsAndFunctions<int>.Multiply = (a, b) => a * b;
    ConstantsAndFunctions<int>.Divide = (a, b) => a / b;
    ConstantsAndFunctions<int>.CreateZero = () => 0;
    ConstantsAndFunctions<int>.CreateOne = () => 1;
    ConstantsAndFunctions<int>.AreEqual = (a, b) => a == b;
    ConstantsAndFunctions<int>.Negate = a => -a;
    ConstantsAndFunctions<int>.IsZero = a => a == 0;
    ConstantsAndFunctions<int>.Copy = a => a;
    ConstantsAndFunctions<int>.Forward = a => a;
}
```

Forward is an essential function. It allows you to only copy a wrapper while
keeping pointer to the same inner data. For example, you have a class
```cs
class Wrapper
{
    private MyData innerData;

    public Wrapper(MyData data) { innerData = data; }
}
```

Then it might look like

```cs
ConstantsAndFunctions<Wrapper>.Copy = a => new Wrapper(innerData.Copy());
ConstantsAndFunctions<Wrapper>.Forward = a => new Wrapper(innerData());
```


## Functionality

Here we list and explain all methods from GT. We use O() syntax to show
asymptotics of an algorithm, where N is a side of a tensor, V is its volume.

### Composition

That is how you work with a tensor's structure:

<details><summary><strong>GetSubtensor</strong></summary><p>

```cs
public GenTensor<T> GetSubtensor(params int[] indecies)
```

Allows to get a subtensor with SHARED data (so that any changes to
intial tensor or the subtensor will be reflected in both).

For example, Subtensor of a matrix is a vector (row).

Works for O(1)
</p></details>

<details><summary><strong>SetSubtensor</strong></summary><p>

```cs
public void SetSubtensor(GenTensor<T> sub, params int[] indecies);
```

Allows to set a subtensor by forwarding all elements from sub to this. Override
ConstantsAndFunctions<T>.Forward to enable it.

Works for O(V)
</p></details>

<details><summary><strong>Transposition</strong></summary><p>

```cs
public void Transpose(int axis1, int axis2);
public void TransposeMatrix();
```

Swaps axis1 and axis2 in this.
TransposeMatrix swaps the last two axes.

Works for O(1)
</p></details>

<details><summary><strong>Concatenation</strong></summary><p>

```cs
public static GenTensor<T> Concat(GenTensor<T> a, GenTensor<T> b);
```

Conatenates two tensors by their first axis. For example, concatenation of
two tensors of shape [4 x 5 x 6] and [7 x 5 x 6] is a tensor of shape
[11 x 5 x 6]. 

Works for O(N)
</p></details>

<details><summary><strong>Stacking</strong></summary><p>

```cs
public static GenTensor<T> Stack(params GenTensor<T>[] elements);
```

Unites all same-shape elements into one tensor with 1 dimension more.
For example, if t1, t2, and t3 are of shape [2 x 5], Stack(t1, t2, t3) will
return a tensor of shape [3 x 2 x 5]

Works for O(V)
</p></details>

<details><summary><strong>Slicing</strong></summary><p>

```cs
public GenTensor<T> Slice(int leftIncluding, int rightExcluding);
```

Slices this into another tensor with data-sharing. Syntax and use is similar to
python's numpy:

```py
v = myTensor[2:3]
```

is the same as

```cs
var v = myTensor.Slice(2, 3);
```

Works for O(N)
</p></details>

### Math operations

That is how you perform mathematical operations on some shapes.
Some operations that are specific to appropriately-shaped tensors
(for example matrix multiplication) are extended to tensors, so if you have
an operation Op, for, say, matrices, it probably has a similar TensorOp that
does the same thing on all matrices of a tensor.

#### Vector operations

<details><summary><strong>Vector dot product</strong></summary><p>

```cs
public static T VectorDotProduct(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> TensorVectorDotProduct(GenTensor<T> a, GenTensor<T> b);
```

Counts dot product of two same-shaped vectors. For example, you have v1 = {2, 3, 4},
v2 = {5, 6, 7}, then VectorDotProduct(v1, v2) = 2 * 5 + 3 * 6 + 4 * 7 = 56.

Works for O(V)
</p></details>

<details><summary><strong>Vector cross product</strong></summary><p>

```cs
public static GenTensor<T> VectorCrossProduct(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> TensorVectorCrossProduct(GenTensor<T> a, GenTensor<T> b);
```

Counts cross product of two same-shaped vectors. The resulting vector is such one
that is perdendicular to all of the arguments.

Works for O(V)
</p></details>

#### Matrix operations

<details><summary><strong>Matrix multiplication</strong></summary><p>

```cs
public static GenTensor<T> MatrixMultiply(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> TensorMatrixMultiply(GenTensor<T> a, GenTensor<T> b);
```

Performs matrix multiplication operation of two matrices. One's height should be the same
as Another's width.

Works for O(N^3)
</p></details>

<details><summary><strong>Determinant</strong></summary><p>

```cs
public T DeterminantLaplace();
public T DeterminantGaussianSafeDivision();
public T DeterminantGaussianSimple();
```

Finds determinant of a square matrix. DeterminantLaplace is the simplest and true
way to find determinant, but it is as slow as O(N!). Guassian elimination works
for O(N^3) but might cause precision loss when dividing. If your type does not
lose precision when being divided, use DeterminantGaussianSimple. Otherwise, for example,
for int, use DeterminantGaussianSafeDivision. 

Works for O(N!), O(N^3)
</p></details>

<details><summary><strong>Inversion</strong></summary><p>

```cs
public void InvertMatrix();
public void TensorMatrixInvert();
```

Inverts A to B such that A * B = I where I is identity matrix.

Works for O(N^4)
</p></details>

<details><summary><strong>Adjugate</strong></summary><p>

```cs
public GenTensor<T> Adjoint();
```

Returns an adjugate matrix.

Works for O(N^4)
</p></details>

<details><summary><strong>Division</strong></summary><p>

```cs
public static GenTensor<T> MatrixDivide(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> TensorMatrixDivide(GenTensor<T> a, GenTensor<T> b)
```

Of A, B returns such C that A == C * B.

Works for O(N^4)
</p></details>

<details><summary><strong>Matrix Power</strong></summary><p>

```cs
public static GenTensor<T> MatrixPower(GenTensor<T> m, int power);
public static GenTensor<T> TensorMatrixPower(GenTensor<T> m, int power);
```

Finds the power of a matrix.

Works for O(log(N) * N^3)
</p></details>

#### Piecewise arithmetics

<details><summary><strong>Tensor and Tensor</strong></summary><p>

```cs
public static GenTensor<T> PiecewiseAdd(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> PiecewiseSubtract(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> PiecewiseMultiply(GenTensor<T> a, GenTensor<T> b);
public static GenTensor<T> PiecewiseDivide(GenTensor<T> a, GenTensor<T> b);
```

Returns a tensor of an operation being applied to every matching pair so that Add is

```
result[i, j, k...] = a[i, j, k...] + b[i, j, k...]
```

Works for O(V)
</p></details>

<details><summary><strong>Tensor and Scalar</strong></summary><p>

```cs
public static GenTensor<T> PiecewiseAdd(GenTensor<T> a, T b);
public static GenTensor<T> PiecewiseSubtract(GenTensor<T> a, T b);
public static GenTensor<T> PiecewiseSubtract(T a, GenTensor<T> b);
public static GenTensor<T> PiecewiseMultiply(GenTensor<T> a, T b);
public static GenTensor<T> PiecewiseDivide(GenTensor<T> a, T b);
public static GenTensor<T> PiecewiseDivide(T a, GenTensor<T> b);
```

Performs an operation on each of tensor's element and forwards them to the result

Works for O(V)
</p></details>

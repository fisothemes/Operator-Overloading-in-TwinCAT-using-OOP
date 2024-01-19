# Operator Overloading in TwinCAT using OOP
# Introduction

Operator overloading in TwinCAT/Codesys systems can be effectively achieved through the application of Object-Oriented Programming (OOP). Before delving into practical examples, it is essential to grasp the foundational concept of operator overloading and its seamless integration with interfaces.

## What is Operator Overloading?

In the realm of programming, operator overloading enables the redefinition of operators' behavior for custom data types. Rather than strictly adhering to predefined operations, operators can be customized to work with user-defined types, enhancing the expressiveness and flexibility of your code.

## Leveraging Interfaces for Operator Overloading

In TwinCAT/Codesys, the synergy of operator overloading and OOP comes to life through the strategic use of interfaces. Interfaces serve as blueprints for classes, defining a set of methods that implementing classes must adhere to. This modularity not only enhances code organization but also facilitates the powerful concept of functional interfaces.

## Functional Interfaces: Enhancing Flexibility

Functional interfaces play a pivotal role in our exploration. These interfaces focus on a specific functionality, in this case, operator overloading. By designing interfaces such as `I_Operator` and `I_Object`, we establish a structured foundation for overloading basic arithmetic operators. This ensures that our classes adhere to a consistent and standardized interface, promoting code clarity and reusability.

## Overloading Arithmetic Operators: Designing Functional Interfaces

Building upon our foundation, let's craft functional interfaces for overloading arithmetic operators in TwinCAT/Codesys. Each interface encapsulates a specific operation, ensuring a clear and modular design.

```js
INTERFACE I_Object EXTENDS __SYSTEM.IQueryInterface
```
```js
INTERFACE I_Operator EXTENDS __SYSTEM.IQueryInterface
```
```js
INTERFACE I_Addition EXTENDS I_Operator
METHOD Plus : I_Addition
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
```
```js
INTERFACE I_Subtraction EXTENDS I_Operator

METHOD Minus : I_Subtraction
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
```
```js
INTERFACE I_Multiplication EXTENDS I_Operator

METHOD Times : I_Multiplication
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
```
```js
INTERFACE I_Division EXTENDS I_Operator

METHOD DivideBy : I_Division
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
```
These interfaces provide a standardized structure for implementing classes to define their behavior during addition, subtraction, multiplication, and division operations. Let's now illustrate these principles with a concrete example involving numeric interfaces.

## Example: Scalars and Vectors
To bring the theoretical concepts into practical application, let's consider the I_Scalar and I_Vector interfaces, representing scalar and vector entities, and their respective implementations in FB_Scalar and FB_Vector.

### Scalar Interface Implementation (FB_Scalar):
The FB_Scalar function block implements the I_Scalar interface, defining methods for converting scalar values to different data types and handling potential errors.

```js
INTERFACE I_Scalar EXTENDS I_Object

METHOD ToF64 : LREAL
VAR_OUTPUT
	e : E_Error;
END_VAR

METHOD ToI64 : LINT
VAR_OUTPUT
	e : E_Error;
END_VAR

METHOD ToU8 : BYTE
VAR_OUTPUT
	e : E_Error;
END_VAR
```
```js
{attribute 'no_explicit_call' := 'do not call this function block directly'}
FUNCTION_BLOCK FB_Scalar IMPLEMENTS I_Scalar, I_Addition, I_Subtraction, I_Multiplication, I_Division
VAR
	_fValue : LREAL; 
END_VAR


-----------------------------------------------------------
METHOD ToF64 : LREAL
VAR_OUTPUT
	e	: E_Error;
END_VAR

ToF64 := THIS^._fValue;

IF TO_STRING(THIS^._fValue) = '#NaN' THEN 
	e := E_Error.NaN;
ELSIF TO_STRING(THIS^._fValue) = '#Inf' THEN
	e := E_Error.PositiveInfinity;
ELSIF TO_STRING(THIS^._fValue) = '-#Inf' THEN
	e := E_Error.NegativeInfinity;
	END_IF


-----------------------------------------------------------
METHOD Plus : I_Addition
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
VAR
	e1, e2 : E_Error;
	ipNumber : I_Scalar;
END_VAR

Plus := THIS^;

IF NOT __QUERYINTERFACE(ipObject, ipNumber) THEN e := E_Error.TypeMismatch; RETURN; END_IF

THIS^._fValue := THIS^.ToF64(e => e1) + ipNumber.ToF64(e => e2);

e := MAX(e1, e2);

-----------------------------------------------------------
METHOD Times : I_Multiplication
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
VAR
	e1, e2 : E_Error;
	ipNumber : I_Scalar;
END_VAR

Times := THIS^;

IF NOT __QUERYINTERFACE(ipObject, ipNumber) THEN e := E_Error.TypeMismatch; RETURN; END_IF

THIS^._fValue := THIS^.ToF64(e => e1) * ipNumber.ToF64( e => e2);

e := MAX(e1, e2);
```

### Vector Interface Implementation (FB_Vector):
Similarly, the FB_Vector function block implements the I_Vector interface, providing methods for vector operations like addition and multiplication.

```js
INTERFACE I_Vector EXTENDS I_Object

METHOD ToArray : ARRAY[0..2] OF LREAL
VAR_INPUT
END_VAR

PROPERTY X : LREAL

PROPERTY Y : LREAL

PROPERTY Z : LREAL
```
```js
{attribute 'no_explicit_call' := 'do not call this function block directly'}
FUNCTION_BLOCK FB_Vector IMPLEMENTS I_Vector, I_Addition, I_Subtraction, I_Multiplication
VAR
	_fX, _fY, _fZ : LREAL;
END_VAR


-----------------------------------------------------------
METHOD Plus : I_Addition
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
VAR
	ipVector : I_Vector;
END_VAR

Plus := THIS^;

IF NOT __QUERYINTERFACE(ipObject, ipVector) THEN e := E_Error.TypeMismatch; RETURN; END_IF

THIS^._fX := THIS^._fX + ipVector.X;
THIS^._fY := THIS^._fY + ipVector.Y;
THIS^._fZ := THIS^._fZ + ipVector.Z;


-----------------------------------------------------------
METHOD Times : I_Multiplication
VAR_INPUT
	ipObject : I_Object;
END_VAR
VAR_OUTPUT
	e : E_Error;
END_VAR
VAR
	ipNumber : I_Scalar;
	ipVector : I_Vector;
	TmpX, TmpY, TmpZ : LREAL;
END_VAR

Times := THIS^;

// Scale vector.
IF __QUERYINTERFACE(ipObject, ipNumber) THEN 
	THIS^.SetValue( THIS^._fX * ipNumber.ToF64(),
					THIS^._fY * ipNumber.ToF64(),
					THIS^._fZ * ipNumber.ToF64());
	RETURN;
	END_IF

// Do the cross-product.
IF __QUERYINTERFACE(ipObject, ipVector) THEN 
	TmpX := THIS^._fX; TmpY := THIS^._fY; TmpZ := THIS^._fZ;
	THIS^.SetValue(	(TmpY*ipVector.Z) - (TmpZ*ipVector.Y),
					(TmpZ*ipVector.X) - (TmpX*ipVector.Y),
					(TmpX*ipVector.Y) - (TmpY*ipVector.X));
	
	RETURN; 
	END_IF

e := E_Error.TypeMismatch;
```

### Main Program Implementation:
Now, let's integrate these function blocks into a main program to showcase the practical usage of operator overloading with Scalars and Vectors.

```js
PROGRAM MAIN
VAR
	bDoubleSquareScal, bAddVec, bCrossProd, bScaleVec: BOOL;
	eError : E_Error;
	fbScalar : FB_Scalar(2.2);
	fbVec1 : FB_Vector(1,2,3);
	fbVec2 : FB_Vector(3,2,1);
END_VAR
============================================================
IF bDoubleSquareScal THEN 
	bDoubleSquareScal := FALSE;
	fbScalar.Times(fbScalar.Plus(fbScalar));
	END_IF

IF bAddVec THEN
	bAddVec := FALSE;
	fbVec1.Plus(fbVec2);
	END_IF
	
IF bCrossProd THEN
	bCrossProd := FALSE;
	fbVec1.Times(fbVec2);
	END_IF
	
IF bScaleVec THEN
	bScaleVec := FALSE;
	fbVec1.Times(fbScalar);
	END_IF
```

In this main program, we create instances of FB_Scalar and two instances of FB_Vector. We then perform various operations such as scalar multiplication, vector addition, vector cross-product, and vector scaling. 

## Conclusion

 Developers can utilize these concepts and implementations to enhance code expressiveness and flexibility in their projects.
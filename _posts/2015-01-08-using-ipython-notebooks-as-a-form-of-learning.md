---
layout: post
category: web micro log
tags:
---

One of the more interesting aspects of using iPython is the ability to combine different languages into one notebook. Below is a session that I was doing as part of
[reddit's dailyprogrammer challenges](http://www.reddit.com/r/dailyprogrammer). As you can see below, notebooks are a great way to comment and display your code in an informative way to "show all working".

I believe that this model is best for when you first begin to learn how to code, especially since it is hard to "visualise" the progress of your code. Now of course when programs go beyond
trivial implementations and are several hundred lines long, this becomes cumbersome and doesn't really make too much sense. But nevertheless it is a great way to take a holistic view of your journey.

---

The goal of this notebook is to demonstrate how we may implement a class in Ruby
and Python to represent a complex number

## Ruby

Firstly we must create a class that represents our complex number

```ruby
%%ruby
class Complex
  def initialize(real, imaginary)
	@real = real
	@imaginary = imaginary
  end
end

a = Complex.new(1,2)
puts a
```

```
	#<Complex:0x3fb1690>
```

But the output above isn't particularly helpful. Let us use string interpolation
in Ruby and override the default `to_s` method.

```ruby
%%ruby
class Complex
  def initialize(real, imaginary)
	@real = real
	@imaginary = imaginary
  end

  def to_s
	"#{@real}+#{@imaginary}i"
  end
end

a = Complex.new(1,2)
puts a

a = Complex.new(1.5,-2)
puts a
```

```
    1 + 2i

    1.5 + -2i
```

Lets add `GetModulus` which returns the modulus of a complex number

```ruby
%%ruby
class Complex
  def initialize(real, imaginary)
	@real = real
	@imaginary = imaginary
  end

  def get_modulus
	Math.sqrt(@real**2 + @imaginary**2)
  end

  def to_s
	"#{@real}+#{@imaginary}i"
  end
end

a = Complex.new(3,4)
p a.get_modulus()
```

```
    5.0
```

Lets now add complex conjugate

```ruby
%%ruby
class Complex
  def initialize(real, imaginary)
	@real = real
	@imaginary = imaginary
  end

  def get_modulus
	Math.sqrt(@real**2 + @imaginary**2)
  end

  def get_conjugate
	Complex.new(@real, -@imaginary)
  end

  def to_s
	"#{@real}+#{@imaginary}i"
  end
end

a = Complex.new(3,4)
p a.get_modulus()
b = a.get_conjugate()
p b.to_s
p b.get_conjugate().to_s
```

```
	5.0

	"3 + -4i"

	"3 + 4i"
```

but now we realise that theres something off in our `to_s` method! We could fix
it, however it still does make mathematical sense.

```ruby
%%ruby
class Complex
  attr_reader :real, :imaginary

  def initialize(real, imaginary)
	@real = real
	@imaginary = imaginary
  end

  def get_modulus
	Math.sqrt(@real**2 + @imaginary**2)
  end

  def get_conjugate
	Complex.new(@real, -@imaginary)
  end

  def to_s
	"#{@real}%si" % sprintf("%+d", @imaginary)
  end
end

a = Complex.new(3,4)
p a.get_modulus
b = a.get_conjugate
puts b.to_s
puts b.get_conjugate.to_s
p b.real
```

```
    5.0

    3-4i

    3+4i

    3
```

```ruby
%%ruby
class Complex
  attr_reader :real, :imaginary

  def initialize(real, imaginary)
	@real = real
	@imaginary = imaginary
  end

  def get_modulus
	Math.sqrt(@real**2 + @imaginary**2)
  end

  def get_conjugate
	Complex.new(@real, -@imaginary)
  end

  def +(num)
	if num.is_a? Complex
	  Complex.new(@real + num.real, @imaginary + num.imaginary)
	elsif num.is_a? Numeric
	  Complex.new(@real + num, @imaginary)
	else
	  raise TypeError
	end
  end

  def -(num)
	if num.is_a? Complex
	  Complex.new(@real - num.real, @imaginary - num.imaginary)
	elsif num.is_a? Numeric
	  Complex.new(@real - num, @imaginary)
	else
	  raise TypeError
	end
  end

  def *(num)
	if num.is_a? Complex
	  Complex.new(@real*num.real - @imaginary*num.imaginary, @imaginary*num.real + @real*num.imaginary)
	elsif num.is_a? Numeric
	  Complex.new(@real*num, @imaginary*num)
	else
	  raise TypeError
	end
  end

  def to_s
	"#{@real}%si" % sprintf("%+d", @imaginary)
  end
end

a = Complex.new(1,2)
b = Complex.new(2,3)

p (a+b).to_s
p (a-b).to_s
p (a*b).to_s
```

```
    "3+5i"

    "-1-1i"

    "-4+7i"
```

## Python

Below is a similar Python implementation of the same problem

```python
class Complex():
	def __init__(self, real, imaginary):
		self.real = real
		self.imaginary = imaginary

	def __str__(self):
		return "%.2f%+.2fi" % (self.real, self.imaginary)



str(Complex(1,2.5))
```

```
    '1.00+2.50i'
```

We can the add the respective parts of it quite easily

```python
class Complex():
	def __init__(self, real, imaginary):
		self.real = real
		self.imaginary = imaginary

	def get_modulus(self):
		return self.real**2 + self.imaginary**2

	def get_conjugate(self):
		return Complex(self.real, - self.imaginary)

	def __str__(self):
		return "%.2f%+.2fi" % (self.real, self.imaginary)


str(Complex(1,2.5).get_conjugate())
```

```
    '1.00-2.50i'
```

```python
Complex(3,4).get_modulus()
```

```
    25
```

Now we can add addition, subtraction, multiplication

```python
class Complex():
	def __init__(self, real, imaginary):
		self.real = real
		self.imaginary = imaginary

	def get_modulus(self):
		return self.real**2 + self.imaginary**2

	def get_conjugate(self):
		return Complex(self.real, - self.imaginary)

	def plus(self, num):
		if isinstance(num, Complex):
			return Complex(self.real + num.real, self.imaginary + num.imaginary)
		elif isinstance(num, (int, long, float)):
			return Complex(self.real + num, self.imaginary)
		else:
			raise TypeError

	def minus(self, num):
		if isinstance(num, Complex):
			return Complex(self.real - num.real, self.imaginary - num.imaginary)
		elif isinstance(num, (int, long, float)):
			return Complex(self.real - num, self.imaginary)
		else:
			raise TypeError

	def times(self, num):
		if isinstance(num, Complex):
			return Complex(self.real * num.real - self.imaginary * num.imaginary,
						   self.imaginary * num.real + self.real * num.imaginary)
		elif isinstance(num, (int, long, float)):
			return Complex(self.real * num, self.imaginary * num)
		else:
			raise TypeError

	def __str__(self):
		return "%.2f%+.2fi" % (self.real, self.imaginary)


print Complex(1,2).plus(Complex(3,4))
print Complex(3,4).minus(Complex(4,5))
print Complex(12,1).times(Complex(1,1))
```

```
    4.00+6.00i
    -1.00-1.00i
    11.00+13.00i
```

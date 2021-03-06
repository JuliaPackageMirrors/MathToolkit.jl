# MathToolkit

[![Build Status](https://travis-ci.org/baruchel/MathToolkit.jl.svg?branch=master)](https://travis-ci.org/baruchel/MathToolkit.jl)

A Julia package providing various functions mainly for the purpose of experimental mathematics (detecting recurrence relations, finding integer relations, etc.).

## Usage

Implemented functions are:

  * `recvec` for finding coefficients of a recurrence relation for a sequence of rational terms;
  * `pslq` for finding an integer relation between several floating-point values.


### Using `recvec` for detecting a recurrence relation

Argument should be a vector containing several `Integer` or `Rational` numbers. The general rule should be to use at least twice as many terms in the argument sequence as the expected length of the resulting vector.

    julia> recvec([1,2,3,4,5,6,7,8,9,10])
    3-element Array{Int64,1}:
      1
     -2
      1

    julia> recvec([0,1,1,2,3,5,8,13])
    3-element Array{Int64,1}:
      1
     -1
     -1

Resulting vector is normalized with all coefficients being integer numbers and the leading coefficient being positive:

    julia> recvec([2^k for k=0:7])
    2-element Array{Int64,1}:
      1
     -2

    julia> recvec([1//2^k for k=0:7])
    2-element Array{Int64,1}:
      2
     -1

#### Checking the result

The following example is a long sequence with a more complicated recurrence relation (of higher order); a check is performed on the last numbers of the sequences (here 16 numbers are involved):

    a = [k^2-2*k^3+5*k-1+k%12 for k=0:64]
    b = recvec(a)
    transpose(a[length(a)-length(b)+1:length(a)])*b

(the final product is 0 meaning the recurrence is right).

#### Result when no recurrence relation is found

An empty vector is returned when no recurrence relation is found:

    julia> recvec([[k for k=1:16];[42]])
    0-element Array{Int64,1}

#### Changing the type used in the computation

The `compute` keyword allows to change the integer type used in the computation; default type is `BigInt` in order to keep the function very general as its default behaviour. Most often it is possible however to use a quicker type in order to increase the speed of the computation:

    julia> recvec([2*k^2-3*k+7 for k=0:7], compute=Int128)
    4-element Array{Int64,1}:
      1
     -3
      3
     -1

#### Changing the type of the resulting vector

The `output` keyword allows to choose the integer type used in the returned value of the function; default type is `Int64`. Many other types may be used according to the needs of the user:

    julia> recvec([3^k-2 for k=0:15], compute=Int128, output=Float64)
    3-element Array{Float64,1}:
      1.0
     -4.0
      3.0

#### Algorithm

The algorithm is a very robust and optimized version of the Padé approximants method. Quicker algorithms can be found for long sequences, but this version is very efficient for most cases. It has a very low memory footprint, even when large sequences are computed.

### Using `pslq` for detecting integer relations

Two arguments are required: a vector containing several floating-point values (the type `BigFloat` should be the type of the variables) and a precision as a floating-point value:

    julia> z=[BigFloat(pi)*2-BigFloat(e)/3-5, BigFloat(pi), BigFloat(e), 1]
    4-element Array{BigFloat,1}:
     3.770913643599047318051909427747849358085897675168917836275666420409406024545445e-01
     3.141592653589793238462643383279502884197169399375105820974944592307816406286198    
     2.718281828459045235360287471352662497757247093699959574966967627724076630353555    
     1.000000000000000000000000000000000000000000000000000000000000000000000000000000    

    julia> pslq(z,1e-10)
    4-element Array{Integer,1}:
      -3
       6
      -1
     -15

    julia> transpose(z)*pslq(z,1e-10)
    1-element Array{BigFloat,1}:
     1.381786968815111140061816298048063931378560058309805021603792555226974688505988e-76

The algorithm has a well-known usage for detecting if a number is algebraic or not:

    julia> x=sin(pi/BigFloat(3))
    8.660254037844386467637231707529361834714026269051903140279034897259665084543988e-01

    julia> pslq([1,x,x^2],1e-20)
    3-element Array{Integer,1}:
     -3
      0
      4

An optional `maxiter` argument (default being 256) allows to change the maximum number of iterations in the main loop of the algorithm.

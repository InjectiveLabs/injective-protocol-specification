## Getting started
We provide VDF (Verifiable Delay Function) Implementations in Golang for two candidate VDFs which are the Piertzak VDF (work in progress) and Sloth (Modular Square Root). You can locate the candidates in
``
vdf/candidates
``.

## Intro to VDF
VDF is a tuple of three algorithms:

<a href="https://www.codecogs.com/eqnedit.php?latex=Setup(\lambda,&space;T)\rightarrow&space;pp" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Setup(\lambda,&space;T)\rightarrow&space;pp" title="Setup(\lambda, T)\rightarrow pp" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=Eval(pp,x)&space;\rightarrow&space;(y,\pi)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Eval(pp,x)&space;\rightarrow&space;(y,\pi)" title="Eval(pp,x) \rightarrow (y,\pi)" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=Verify(pp,x,y,\pi)&space;\rightarrow&space;\{accept,reject\}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Verify(pp,x,y,\pi)&space;\rightarrow&space;\{accept,reject\}" title="Verify(pp,x,y,\pi) \rightarrow \{accept,reject\}" /></a>

### Modular Square Root

#### Evaluation
Sloth, or modular sqaure root, is one of the simplest yet effective candidate for verifiable delay function.

Given a prime field ``p``, starting value ``x``, and delay parameter ``t``:

We assume that finding ``x^2 % p`` is much easier than finding ``sqrt(x) % p``.
This is because the fastest existing algorithm to find modular square root of ``x``
is ``x^((p+1)/4) % p``, which requires a lot more operations than simply squaring ``x``.

This also implies that as the prime field ``p`` gets larger, more operations are required.

To implement Sloth, we first select a prime field ``p`` where ``p%4=3``. When we evaluate starting value ``x``, 
we have to first check whether ``x`` is a quadratic residue by checking whether 
``a^((p-1)/2)%p=1``. If not, we simply change starting value ``x`` to ``(-x)%p``.

#### Chaining

In Sloth, the ``t`` parameter specifies the number of iterations the VDF evaluation has undergone.
Since ``sqrt(x)->y``, we can extend the delay of Sloth by performing the same evaluation over ``y`` with 
``sqrt(x)-> y , sqrt(y)-> y_1 , sqrt(y_1)->y_2 ...``. This way long as the prime field (>256 bits)is large enough, 
the length of delay provided from adjusting ``t`` should be more than enough.

 

#### Proof Generation
Sloth is fairly straightforward when it comes to verification. We do not need to submit a proof 
for a basic Sloth implementation. Verifiers can simply square ``y`` once or ``y_t`` t+1 times to 
acquire the starting value ``x``.

For a faster verification, we can use SNARK proofs to be submitted along with ``y``. 
However, if we want better a verification vs. evaluation gap, there are better candidates 
for it. 

### Time Proof
In our protocol,``t`` is the time parameter for proof of elapsed time. In the context of Sloth, ``t`` 
is the number of modular square root evaluation. ``t`` can be computed incrementally for it to become a time proof.


### Interaction

Our client will be interacting with the VDF candidates under through `vdf_interface.go`. The exported functions are:
```
Sloth_fixed_delay(p_parameter string, starting_value string, iteration string) string 
Sloth_eval(p_parameter string, starting_value string, iteration string) string  
Sloth_verify(p_parameter string, starting_value string, iteration string, ending_value string) bool 
```
We created `Sloth_fixed_delay` for testing purposes. 

`Sloth_eval` takes in:

prime number: `p_parameter : type string`
`x` or starting value: `starting_value: type string`
`t: type string` iteration count: `iteration: type string`

and outputs

`y_t` or ending value at iteration `t`: `ending_value: type string`

`Sloth_verify` takes in:

prime number: `p_parameter : type string`
`x` or starting value: `starting_value: type string`
`t: type string` iteration count: `iteration: type string`
`y_t` or ending value at iteration `t`: `ending_value: type string`

and outputs

the result of the verification:  `resulte : type bool`

Within this file, we have stored a few prime numbers for demonstration purposes, these numbers will not be in production.

`vdf_interface` uses the candidates in `\candidates` subdirectory. Currently we are only using `sloth.go`. In `sloth.go`, the relevant functions exported are:
```
Eval( args[3]string ) string
Verify( args[4]string ) bool
```

Recalling [Intro To VDF](##intro-to-vdf), the `string` arguments for `Eval` are `[ p , x , t ]` and the function returns `y_t` as the output. This function will evaluate the VDF using the Sloth candidate with the ending value `y_t` as the output. 

`Verify` arguments are ` [ p , x , t , y ] ` and the function returns a boolean as the output. This function takes all the VDF parameters and returns whether the ending value `y_t` was correct.





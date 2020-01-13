# Miscellaneous

## Front-Running Prevention

## Verifiable Delay Function

### Implementation

In our protocol,$$t$$ is the time parameter for proof of elapsed time. In the context of Sloth, `t` is the number of modular square root evaluation. `t` can be computed incrementally for it to become a time proof.

After submitting the take order information adhering to 0x schema, the user will generate `x` value by creating the `orderHash` from `Order`.

Using `x`, user begins to calculate VDF \(Sloth for example\): `sqrt(x)-> y , sqrt(y)-> y_1 , sqrt(y_1)->y_2 ...` until it reaches a satisfactory checkpoint `y_t`. User then submits `y_t, t , x , order_Hash` to a relayer so that it can map them to the order information the user submitted previously. The relayer will then perform `vdf_verify`. to ensure the validity of the user's VDF proof. In sloth's case, the relayer can simply calculate `((y_t)^2)^(t+1)==x`.

After `Vdf_verify`, the relayer will append `t` to the user's existing order information. Since traders are allowed to submit multiple transactions for time proofs \(ex: `tx_1: {y_t , t ,x} tx_2: {y_2t , 2t , x} tx_3: {y_3t , 3t , x}`\), the relayer should also check if the `t` is bigger than the existing `t` attached to the trader's information before performing any VDF verification.

Before a block is mined, the relayers match take orders with make orders on a `t`-priority basis. Meaning that for each make order, the taker orders are sorted by descending `t` and filled in that order.

### Interaction

Our client will be interacting with the VDF candidates under through `vdf_interface.go`. The exported functions are:

```text
Sloth_fixed_delay(p_parameter string, starting_value string, iteration string) string 
Sloth_eval(p_parameter string, starting_value string, iteration string) string  
Sloth_verify(p_parameter string, starting_value string, iteration string, ending_value string) bool
```

We created `Sloth_fixed_delay` for testing purposes.

`Sloth_eval` takes in:

prime number: `p_parameter : type string` `x` or starting value: `starting_value: type string` `t: type string` iteration count: `iteration: type string`

and outputs

`y_t` or ending value at iteration `t`: `ending_value: type string`

`Sloth_verify` takes in:

prime number: `p_parameter : type string` `x` or starting value: `starting_value: type string` `t: type string` iteration count: `iteration: type string` `y_t` or ending value at iteration `t`: `ending_value: type string`

and outputs

the result of the verification: `resulte : type bool`

Within this file, we have stored a few prime numbers for demonstration purposes, these numbers will not be in production.

`vdf_interface` uses the candidates in `\candidates` subdirectory. Currently we are only using `sloth.go`. In `sloth.go`, the relevant functions exported are:

```text
Eval( args[3]string ) string
Verify( args[4]string ) bool
```

The `string` arguments for `Eval` are `[ p , x , t ]` and the function returns `y_t` as the output. This function will evaluate the VDF using the Sloth candidate with the ending value `y_t` as the output.

`Verify` arguments are `[ p , x , t , y ]` and the function returns a boolean as the output. This function takes all the VDF parameters and returns whether the ending value `y_t` was correct.


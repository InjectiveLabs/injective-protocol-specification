# Miscellaneous

## Front-Running Prevention

### Soft-Cancels

### Selective Delay

### Verifiable Delay Function

In our protocol,$$t$$ is the time parameter for proof of elapsed time, which in the context of Sloth, is the number of modular square root evaluations. $$t$$ can be computed incrementally for its use as a time proof.

After submitting the take order information adhering to 0x schema, the user will generate $$x$$ value by creating the $$orderHash$$ from $$Order$$.

Using $$x$$, the user begins to calculate VDF \(Sloth for example\): $$y \leftarrow msqrt(x), y_1 \leftarrow msqrt(y), y_2 \leftarrow msqrt(y_1)$$ where $$msqrt(x) = x^{ \frac{(p + 1)}{4}} \mod p$$ $$t$$ times until the user thus obtains $$y_t$$. The user then submits $$(y_t, t , x)$$ to a relayer. The relayer will then perform $$vdf_{verify}$$. to ensure the validity of the user's VDF proof. In sloth's case, the relayer can simply verify that $$((y_t)^2)^{(t+1)}=x$$.

After $$vdf_{verify}$$, the relayer will append $$t$$ to the user's existing order information. Since traders are allowed to submit multiple transactions for time proofs \(ex: $$tx_1: \{y_t , t ,x\} tx_2: \{y_2t , 2t , x\} tx_3: \{y_3t , 3t , x\}$$\), the relayer should also check if the $$t$$ is bigger than the existing $$t$$ attached to the trader's information before performing any VDF verification.

Before a block is mined, the relayers match take orders with make orders on a $$t$$-priority basis. Meaning that for each make order, the taker orders are sorted by descending $$t$$ and filled in that order.

### Interaction

Our client will be interacting with the VDF candidates under through `vdf_interface.go`. The exported functions are:

```text
Sloth_fixed_delay(p_parameter string, starting_value string, iteration string) string 
Sloth_eval(p_parameter string, starting_value string, iteration string) string  
Sloth_verify(p_parameter string, starting_value string, iteration string, ending_value string) bool
```

We created `Sloth_fixed_delay` for testing purposes.

`Sloth_eval` takes in:

Prime number $$p$$, starting value $$x$$, and $$t$$ iteration count and outputs $$y_t$$ or ending value at iteration $$t$$.

$$Sloth_{verify}$$ takes in Prime number $$p$$, starting value $$x$$, iteration count $$t$$, ending value $$y_t$$ and outputs the boolean result of the verification.

Within this file, we have stored a few prime numbers for demonstration purposes, these numbers will not be in production.

`vdf_interface` uses the candidates in `\candidates` subdirectory. Currently we are only using `sloth.go`. In `sloth.go`, the relevant functions exported are $$Eval(p, x, t)$$ which evaluates the VDF using the Sloth candidate and returns $$y_t$$ and $$Verify(p, x, t, y)$$ which verifies the validity of the VDF output based off the starting parameters and returns $$true/false$$.


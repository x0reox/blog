# random

the `rand()` standard library function generates a pseudo-random number based on the initial seed value
given with `srand()`. it is pseudo random because the algorithm used is still deterministic—the same input
always converts to the same output. if a seed value is not explicitly set, `rand()` is automatically
given a default seed of 1. with this in mind, see why the following program uses `rand()` ineffectively:

```{.c .numberLines}
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xcafebabe ){
		printf("Good!\n");
		setregid(getegid(), getegid());
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}

```

the value given to `random` is not really random—we can emulate the behavior in
our own program:

```{.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main()
{
        unsigned int random = rand();
        printf("%d\n", random);
}
```

```markdown
random@ubuntu:/tmp/random$ ./a.out
1804289383
random@ubuntu:/tmp/random$ ./a.out
1804289383
random@ubuntu:/tmp/random$ ./a.out
1804289383
```

as you can see, we get the same number every time.

`^` is the bitwise XOR operator. it compares two values bit-by-bit and returns a new value
based these rules: return a 1 if the two bits are different, and return a 0 if the two bits are the same.
for example:

```markdown
5 ^ 3 = 6

  0101  (5)
^ 0011  (3)
-----
  0110  (6)
```

the key check here is `key ^ random == 0xcafebabe`. we already know the values of `random` and `0xcafebabe`:

```markdown
random@ubuntu:/tmp/random$ export random=$(echo "obase=2;$(./a.out)" | bc)
random@ubuntu:/tmp/random$ echo $random
1101011100010110100010101100111
random@ubuntu:/tmp/random$ export cafebabe=$(echo "obase=2;$(echo $((0xcafebabe)))" | bc)
random@ubuntu:/tmp/random$ echo $cafebabe
11001010111111101011101010111110
```

so we can just solve for `key` by comparing each bit. I opted to automate this with python:

```python
import os

random = os.environ["random"]
cafebabe = os.environ["cafebabe"]
random = "0" + random
key = ""
for i in range(len(random)):
    if random[i] != cafebabe[i]:
        key += "1"
    else:
        key += "0"
print(int(key, 2))
```

and verify:

```markdown
random@ubuntu:~$ key=$(python3 /tmp/random/xor.py)
random@ubuntu:~$ printf '0x%x\n' $(( $key ^ $(/tmp/random/a.out) ))
0xcafebabe
```

```markdown
random@ubuntu:~$ python /tmp/random/xor.py | ./random
Good!
m0mmy_I_can_predict_rand0m_v4lue!
```

note that `bc` drops the leading 0 for `random`, so I added it back to compare the two values correctly.

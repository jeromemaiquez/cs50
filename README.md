# Z-TEST
#### Description:

Z-TEST is a program that performs a one-sample z-test for you.
It does this by creating a class `Z_Test` with the following **instance variables**:

##### Required arguments:
| Name | Flags | Description | Type |
| ---- | ----- | ----------- | ---- |
| `smean` | `-m` , `--smean` | sample mean | `float` |
| `pmean` | `-p` , `--pmean` | population mean | `float` |
| `sigma` | `-s` , `--sigma` | pop. standard deviation | `float` |
| `size` | `-n` , `--size` | sample size | `int` |

##### Optional arguments:
| Name | Flags | Description | Type | Default |
| ---- | ----- | ----------- | ---- | ------- |
| `alpha` | `-a` , `--alpha` | significance level | `float` | `0.05` |
| `tail` | `-t` , `--tail` | choice of tail | `str` | `"T"` |
 
`Z_Test` also has the following **instance methods**, one for each step of the z-test process:

1. `state_null()`

   - returns an f-string stating the null hypothesis
   
   - takes `smean` and `pmean`
   
   ```

   def state_null(self):
       return f"Null (H0)          : {self.smean} = {self.pmean}"

   ```
   
2. `state_alternate()`
   
   - takes `smean`, `pmean`, and `tail`
   
   - chooses an `operator` based on `tail`

     - `tail == "L"` --> `operator = "<"`
     - `tail == "R"` --> `operator = ">"`
     - `tail == "T"` --> `operator = "!="`

   - returns an f-string stating the alternate hypothesis

   ```

   def state_alternate(self):
       if self.tail == "R":
           operator = ">"
       elif self.tail == "L":
           operator = "<"
       elif self.tail == "T":
           operator = "!="
       else:
           sys.exit("Invalid choice of tails")
       return f"Alternate (H1)     : {self.smean} {operator} {self.pmean}"

   ```

3. `get_crit_val()`
   - calculates critical value via `NormalDist().inv_cdf(p)` in `statistics` module
   - value of `p` differs depending on `tail`:
     - `tail == "L"` -->  `p = alpha`
     - `tail == "R"` -->  `p = 1 - alpha`
     - `tail == "T"` -->  `p = 1 - (alpha / 2)`
   
   - returns `crit_val` rounded to 3 decimal places
   
    ```
    def get_crit_val(self):
        if self.tail == "T":
            p = 1 - (self.alpha / 2)
        elif self.tail == "R":
            p = 1 - self.alpha
        elif self.tail == "L":
            p = self.alpha
        else:
            sys.exit("Invalid choice of tail")

        return round(NormalDist().inv_cdf(p), 3)
    ```

4. `get_z_score()`
   - calculates z-score for one-sample z-test using the formula:

     ![Formula for z-score in one-sample z-test, with definitions for each parameter.](https://miro.medium.com/v2/resize:fit:574/format:webp/1*b7iZyQyP8SJ-W51x_L5ekg.png)
   - returns `z_score` rounded to 3 decimal places

    ```
    def get_z_score(self):
        try:
            z = (self.smean - self.pmean) / (self.sigma / sqrt(self.size))
            return round(z, 3)
        except (ZeroDivisionError, ValueError, TypeError):
            sys.exit("Invalid sample size")
    ```

5. `reject_null()`
   - first part checks whether `z_score` is outside of acceptance region:

     - `tail == "L"` -->  `z_score < crit_val`
     - `tail == "R"` -->  `z_score > crit_val`
     - `tail == "T"` -->  `z_score > crit_val`
   - then returns `"Rejected"` if `True`, `"Not Rejected"` if `False`

   ```
    def reject_null(self):
        if self.tail == "L":
            z_or_crit = self.get_z_score() < self.get_crit_val()
        else:
            z_or_crit = self.get_z_score() > self.get_crit_val()

        if z_or_crit:
            return "Rejected"
        else:
            return "Not Rejected"
   ```

Additional instance methods handle checks and printing of z-test results:

1. `check_size()` : exits if `size < 30`

   ```
   def check_size(self):
    if self.size <= 0:
        sys.exit("Invalid sample size")
    elif 0 < self.size < 30:
        sys.exit("Sample size too small")
   ```
  
2. `run_test()` : runs all previous `Z_Test()` instance methods, then prints them in f-strings

   ```
   def print_result(self):
       self.check_size()
       print(self.state_null())
       print(self.state_alternate())
       if self.tail == "T":
           print(f"Critical           : +-{self.get_crit_val()}")
       else:
           print(f"Critical           : {self.get_crit_val()}")
       print(f"Z-Score            : {self.get_z_score()}")
       print(f"Null Hypothesis is : {self.reject_null()}")
   ```

Whenever a `Z_Test()` object is initialized, the command-line arguments are automatically
parsed into a dictionary, whose values are then assigned by key into the corresponding instance variables.

```
def __init__(self):
    size_choices = [x / 100 for x in range(0, 100)]
    tail_choices = ("L", "R", "T")
    parser = argparse.ArgumentParser(description="Perform z-test")
    parser.add_argument("-m", "--smean", type=float, required=True, help="Sample mean")
    parser.add_argument("-p", "--pmean", type=float, required=True, help="Population mean")
    parser.add_argument("-s", "--sigma", type=float, required=True, help="Population standard deviation")
    parser.add_argument("-n", "--size", type=int, required=True, help="Sample size: must be >=30")
    parser.add_argument("-a", "--alpha", default=0.05, type=float, choices=size_choices, help="Significance level")
    parser.add_argument("-t", "--tail", default="T", type=str, choices=tail_choices, help="Left (L), right (R), or two (T) tails")

    args = vars(parser.parse_args())

    self.smean = args["smean"]
    self.pmean = args["pmean"]
    self.sigma = args["sigma"]
    self.size = args["size"]
    self.alpha = args["alpha"]
    self.tail = args["tail"]
```

Since all related functionality is encapsulated within the `Z_Test` class, all that is needed in `main()` for
the program to run is an instantiation of a `Z_Test()` object, then to run the `run_test()` method on it.

```
def main():
    z = Z_Test()
    z.run_test()
```

Once `project.py` is run with the following arguments...

```
python project.py -m 107 -p 100 -s 15 -n 30 -t R -a 0.05
```

...here is the corresponding output:

```
Null (H0)          : 107.0 = 100.0
Alternate (H1)     : 107.0 > 100.0
Critical           : 1.645
Z-Score            : 2.556
Null Hypothesis is : Rejected
```

#### Design Choices:
From my experience with GDAL, I learned how many Python programs take in arguments from the command line 
instead of having to prompt the user. I wanted to do this for my Z-Test program as well, because it is so 
much easier to run and test the program multiple times. I am happy to have learned about the `argparse` 
module, and I'll be sure to use this in more of Python programs from here on out.

I also wanted to implement `Z_Test` as a class, envisioning it as an entity with attributes (e.g. `smean`) 
and actions (e.g. `get_z_score()`). I wanted to encapsulate as much related functionality as possible within
this class, such that the code remaining in `main()` is minimal. I wanted to see how far I can extend this
encapsulation, with the hopes that the `Z_Test` class is as reusable as possible.

I also considered providing the user a choice in performing either a one-sample or a two-sample z-test. 
I can probably implement this given enough time, but since I was intimidated by the sheer number of 
additional parameters in two-sample z-tests, and I was working on this project in the dead of night, 
I decided against this. I'm sure smarter people have already done this better.

But why a z-test at all? I wanted to turn into code a process I was already familiar with, yet which I knew 
took a lot of steps. This project was also an excuse to brush up on some basics of statistics and hypothesis 
testing, which is a good foundation to have as I build up my proficiency as a geospatial professional.

#### Files:
- `z_test.py` : main file, containing all the functions
- `test_z_test.py` : unit test for `z_test.py` (Not yet implemented)

#### Modules:
All modules imported are part of the Python Standard Library.
- `argparse` : parser for command-line arguments and options
- `sys` : allows exit from program when errors are raised
- `statistics` : allows access to NormalDist() class methods
- `math` : allows access to sqrt() function

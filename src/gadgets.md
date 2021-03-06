[<img width=900 src="https://raw.githubusercontent.com/txt/mase/master/img/banner1.png">](https://github.com/txt/mase/blob/master/README.md)   
[At a glance...](https://github.com/txt/mase/blob/master/OVERVIEW.md) |
[Syllabus](https://github.com/txt/mase/blob/master/SYLLABUS.md) |
[Models](https://github.com/txt/mase/blob/master/MODELS.md) |
[Code](https://github.com/txt/mase/tree/master/src) |
[Lecturer](http://menzies.us) 





# GADGETS: Timm's Generic Optimizer Gadgets (and Goodies)

<img width=400 align=right src="../img/gogo.jpg">


`Gadgets` are a library of utilities for working with
optimization models. 

With `Gadgets`, it is possible to encode something
that looks a little like....

```python
for model in [Schaffer,Fonseca,...]:
  for optimizer in [sa,mws...]:
    for _ in xrange(20):
      optimizer(model())
```

Note that:

+ For samples of how to use this code, see [gadgetsok.py](gadgetsok.py).
+ In the following, anything in `this font` is some method or class in the code.

## What are `Gadgets`?

`Gadgets` are a _Factory_ and a _Facade_:

+ In class-based programming, the _factory_ method
  pattern is a _creational pattern_ which uses factory
  methods to deal with the problem of creating
  objects without specifying the exact class of
  object that will be created. This is done by
  creating objects via (e.g.)  calling a factory
  method-- either specified in an interface and
  implemented by child classes.
+ A _facade_ is an object that provides a simplified 
  interface to a larger body of code, such as a class library. A facade can:
   + make a software library easier to use,
     understand and test, since the facade has
     convenient methods for common tasks;
   + make the library more readable, for the same
     reason; 
   + reduce dependencies of outside code on
     the inner workings of a library, since most
     code uses the facade, thus allowing more
     flexibility in developing the system;

`Gadgets` are a _facade_ since, every time I code a new optimizer, its always some mix-and-match of
numerous lower-level facilities. 

+ On Sundays and Wednesdays, I think optimizers should inherit from
`Gadgets`.
+ On every other day, I believe that since not every gadget applies to every optimizer,
its just more flexible and easier to place all those lower-level facilities into a box, and just 
call interface methods to that box.

_Factories_ assemble _parts_. In this code, I know of three kinds of parts:

+ Parts to hold the specific values from one set of
  decisions, objectives and (optionally) some aggregation of the objective
  scores. In the following, these will be called
  + `decs`
  + `objs`
  + `aggregate`
+ Parts for the some _logging code_ that keeps track of the observed `decs` and `objs, aggregate` values.
+ Parts that talk `About` legal values for the `decs` and `objs` and, for `objs` if we want to minimize
  or maximize those scores.

## Parts of the `Gadgets`

### `Candidate`s

`Candidate`s objects have  objectives, decisions and maybe some aggregate value. 
Using a factory method,  we will fill in `Candidate`s with either 

+ The specific values from one set `decs` and `objs, aggregate`;
+ Or, `Log`ging objects that remember what specific values were ever assigned to  `decs` and `objs, aggregate`;
+ Or, `About` objects that defined what are legel values from the specific values  
  (and, for `objs` if we want to minimize
  or maximize those scores)

In the following, I sometimes refer to `Candidate` objects as `can`. For example, the `ok` method
received a `can` and returns true if the current contents of `decs` are valid.

<a href="gadgets.py#L141-L185"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

   1:   class Candidate(object):
   2:     def __init__(i,decs=[],objs=[]):
   3:       i.decs,i.objs=decs,objs
   4:       i.aggregate=None
   5:       i.abouts = i.about()
   6:       
   7:     def __repr__(i):
   8:       "Present me in a  string"
   9:       return printer(i,decs=i.decs,
  10:                      objs=i.objs,
  11:                      aggregated=i.aggregate)
  12:   
  13:     def __getitem__(i,key):
  14:       "Simple way to access decs or objs or aggregates."
  15:       return i.__dict__[key]
  16:     
  17:     def ok(i,can):
  18:       "Maybe overwritten by subclass."
  19:       return True
  20:     
  21:     def about(i):
  22:       """Factory method for return a Candidate full of 
  23:          About objects."""
  24:       assert False,'implemented by subclass'
  25:       
  26:     
  27:     def clone(i,what = lambda _: None):
  28:       """A genetic factory that makes a new  thing
  29:          like receiver, filled in with 'what' objects."""
  30:       j      = object.__new__(i.__class__)
  31:       j.decs = [what(x) for x in i.decs]
  32:       j.objs = [what(x) for x in i.objs]
  33:       j.aggregate = what(i.aggregate)
  34:       j.abouts = i.abouts
  35:       return j
  36:     
  37:     def alongWith(i,j=None):
  38:       "Convenient iterator."
  39:       if j:
  40:         for one,two in zip(i.decs, j.decs):
  41:           yield one,two
  42:         for one,two in zip(i.objs, j.objs):
  43:           yield one,two
  44:         yield i.aggregate, j.aggregate
  45:         
```

Using the above, we can build a _factory_ method
called `about` that returns what we know `About`
each candidate.

#### Schaffer

One decision, two objectives, zero  constraints.

<a href="gadgets.py#L197-L207"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

  46:   class Schaffer(Candidate):
  47:     def about(i):
  48:       def f1(can):
  49:         x = can.decs[0]
  50:         return x**2
  51:       def f2(can):
  52:         x = can.decs[0]
  53:         return (x-2)**2
  54:       i.decs = [An("x",   lo = -10**5, hi = 10**5)]
  55:       i.objs = [Less("f1",  maker=f1),
  56:                 Less("f2", maker=f2)]
```

In the above, `An` and `Less` are really `About` objects that define legal ranges for values 
 (and, for `objs` if we want to minimize
  or maximize those scores).

Note also that `f1` and `f2` are nested methods that accepted a `Candidate` object (which,
you will recall, I call `can`s).


#### Fonseca

Three decisions, two objectives, zero constraints.

<a href="gadgets.py#L223-L236"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

  57:   class Fonseca(Candidate):
  58:     n=3
  59:     def about(i):
  60:       def f1(can):
  61:         z = sum([(x - 1/sqrt(Fonseca.n))**2 for x in can.decs])
  62:         return 1 - ee**(-1*z)
  63:       def f2(can):
  64:         z = sum([(x + 1/sqrt(Fonseca.n))**2 for x in can.decs])
  65:         return 1 - ee**(-1*z)
  66:       def dec(x):
  67:         return An(x, lo=-4, hi=4)
  68:       i.decs = [dec(x) for x in range(Fonseca.n)]
  69:       i.objs = [Less("f1",  maker=f1),
  70:                 Less("f2",  maker=f2)]
```

Note the use of a list comprehension to create multiple decisions, all with similar properties.
This is handy here and, for more complex models like `ZDT1` with 30 decisions with similar properties,
it is very useful indeed.

#### Kursawe

Three decisions, two objectives,  zero constraints.

<a href="gadgets.py#L248-L262"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

  71:   class Kursawe(Candidate):
  72:     n=3
  73:     def about(i,a=1,b=1):
  74:       def f1(can):
  75:         def xy(x,y):
  76:           return -10*ee**(-0.2*sqrt(x*x + y*y))
  77:         a,b,c = can.decs
  78:         return xy(a,b) + xy(b,c)
  79:       def f2(can):
  80:         return sum( (abs(x)**a + 5*sin(x)**b) for x in can.decs )
  81:       def dec(x):
  82:         return  An(x, lo=-5, hi=5)           
  83:       i.decs = [dec(x) for x in range(Kursawe.n)]
  84:       i.objs = [Less("f1",  maker=f1),
  85:                 Less("f2",  maker=f2)]
```

#### ZDT1

Thirty decisions, two objectives,  zero constraints.

<a href="gadgets.py#L270-L282"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

  86:   class ZDT1(Candidate):
  87:     n=30
  88:     def about(i):
  89:       def f1(can):
  90:         return can.decs[0]
  91:       def f2(can):
  92:         g = 1 + 9*sum(x for x in can.decs[1:] )/(ZDT1.n-1)
  93:         return g*abs(1 - sqrt(can.decs[0]*g))
  94:       def dec(x):
  95:         return An(x,lo=0,hi=1)
  96:       i.decs = [dec(x) for x in range(ZDT1.n)]
  97:       i.objs = [Less("f1",maker=f1),
  98:                 Less("f2",maker=f2)]
```

Again, note the use of a list comprehension to create multiple decisions, all with similar properties.

#### Viennet4

Two decisions, three objectives,  three constraints (all codes into the `ok` method).

<a href="gadgets.py#L292-L315"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

  99:   class Viennet4(Candidate):
 100:     n=2
 101:     def ok(i,can):
 102:        one,two = can.decs
 103:        g1 = -1*two - 4*one + 4
 104:        g2 = one + 1            
 105:        g3 = two - one + 2
 106:        return g1 >= 0 and g2 >= 0 and g3 >= 0
 107:     def about(i):
 108:       def f1(can):
 109:         one,two = can.decs
 110:         return (one - 2)**2 /2 + (two + 1)**2 /13 + 3
 111:       def f2(can):
 112:         one,two = can.decs
 113:         return (one + two - 3)**2 /175 + (2*two - one)**2 /17 - 13
 114:       def f3(can):
 115:         one,two= can.decs
 116:         return (3*one - 2*two + 4)**2 /8 + (one - two + 1)**2 /27 + 15
 117:       def dec(x):
 118:         return An(x,lo= -4,hi= 4)
 119:       i.decs = [dec(x) for x in range(Viennet4.n)]
 120:       i.objs = [Less("f1",maker=f1),
 121:                 Less("f2",maker=f2),
 122:                 Less("f3",maker=f3)]
```

### `Log`ging Objects 

Another kind of part that is assembled into a `Candidate` by a factory methods are
`Log` objects. These  remembers the range of values
seen so far.

Note one small details about these `Log`s:

+ Sometimes we are logging information
  about one run within other runs. So `Log` has an `also` pointer
  which, if non-nil, is another place to repeat the same information. 

<a href="gadgets.py#L331-L357"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 123:   class Log:
 124:     def __init__(i,init=[],also=None):
 125:       i.n,i.lo, i.hi, i.also, i._some= 0,None, None, also,Some()
 126:       map(i.__add__,init)
 127:     def adds(i,lst):
 128:       map(i.__add__,lst)
 129:     def __add__(i,x):
 130:       i.n += 1
 131:       if   i.empty() : i.lo = i.hi = x # auto-initialize
 132:       elif x > i.hi     : i.hi = x
 133:       elif x < i.lo     : i.lo = x
 134:       if i.also:
 135:         i.also + x
 136:       i._some += x     # NOTE1
 137:       return x
 138:     def some(i):
 139:       return i._some.any
 140:     def tiles(i,tiles=None,ordered=False,n=3):
 141:       return r3(ntiles(i.some(),tiles,ordered),n)
 142:     def empty(i):
 143:       return i.lo == None
 144:     def norm(i,x):
 145:       return (x - i.lo)/(i.hi - i.lo + 10**-32)
 146:     def stats(i,tiles=[0.25,0.5,0.75]):
 147:       return ntiles(sorted(i._some.any),
 148:              ordered=False, 
 149:              tiles=tiles)
```
 
_NOTE1_ As a side-effect of logging, we also keep a small sample of
  the logged items This will come in handy... later. The code
for keeping _Some_ values is shown below.
   
<a href="gadgets.py#L365-L381"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 150:   @setting
 151:   def SOMES(): return o(
 152:       size=256
 153:       )
 154:   
 155:   class Some:
 156:     def __init__(i, max=None): 
 157:       i.n, i.any = 0,[]
 158:       i.max = max or the.SOMES.size
 159:     def __iadd__(i,x):
 160:       i.n += 1
 161:       now = len(i.any)
 162:       if now < i.max:    
 163:         i.any += [x]
 164:       elif r() <= now/i.n:
 165:         i.any[ int(r() * now) ]= x 
 166:       return i
```

### `About` Objects 

The `About` class (and its variants: `Less` and `More`) define our expectation for 
each `Candidate`
values.

If we need a value for a `Candidate`, we call `.maker()`:

+ For decisions,
  this just pulls a value from the known `hi` and `lo` ranges;
+ For objectives,
  call some `maker` function passed over an initialization time.

Note that `About` is a handy place to implement some useful services:

+ Checking if a value is `ok` (in bounds `lo..hi`);
+ `restrain`ing out of bound values back to `lo..hi`;
+ `wrap`ing out of bounds value via modulo;
+ How to compute the distance `fromHell`.

<a href="gadgets.py#L405-L431"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 167:   def lt(i,j): return i < j
 168:   def gt(i,j): return i > j
 169:   
 170:   class About(object):
 171:     def __init__(i, txt, init=None,
 172:                     lo=-10**32, hi=10**32,
 173:                     better=lt,
 174:                     maker=None):
 175:       i.txt,i.init,i.lo,i.hi = txt,init,lo,hi
 176:       i.maker = maker or i.guess
 177:       i.better= better
 178:     def __repr__(i):
 179:       return 'o'+str(i.__dict__)
 180:     def guess(i):
 181:       return i.lo + r()*(i.hi - i.lo)
 182:     def restrain(i,x):
 183:       return max(i.lo, min(i.hi, x))
 184:     def wrap(i,x):
 185:       return i.lo + (x - i.lo) % (i.hi - i.lo)
 186:     def norm(i,x):
 187:       return (x - i.lo) / (i.hi - i.lo + 10**-32)
 188:     def ok(i,x):
 189:       return i.lo <= x <= i.hi
 190:     def fromHell(i,x,log,min=None,max=None):
 191:       norm = i.norm if log.lo == None else log.norm
 192:       hell = 1 if i.better == lt else 0
 193:       return (hell - norm(x)) ** 2
```

Note that many of the above will be called many times as we (e.g.) fill in the decisions
of a `can` (e.g. `guess`, `wrap`, `norm`) or its objectives (e.g. `fromHell`).
 
Using the above, we can succinctly specify objectives
that want to minimize or maximize their values.

<a href="gadgets.py#L441-L444"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 194:   A = An = Less = About
 195:   
 196:   def More(txt,*lst,**d):
 197:     return About(txt,*lst,better=gt,**d)
```

## The `Gadgets` Facade

Note that `Gadgets` stores most of the generic processing
of my optimizers. Hence the control params of `Gadgets`
is really the control params of most of the optimization.

<a href="gadgets.py#L454-L464"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 198:   @setting
 199:   def GADGETS(): return  o(
 200:       baseline=50,
 201:       era=50,
 202:       mutate = 0.3,
 203:       epsilon=0.01,
 204:       lives=5,
 205:       verbose=True,
 206:       nudge=1,
 207:       patience=64
 208:   )
```

```
g = Gadgets(Schaffer())
```

(Note the brackets-- this creates a new instance.)

Here is the `Gadgets` facade. Note that it offers a wide range of services
including:

+ Factory methods (for generating empty `can`s, or `can`s filled with `Log`s.)
+ Logging methods (for remembering what values were generated)
+ Methods for filling in decisions;
+ Methods for filling in objectives;
+ Mutation methods;
+ Methods for fully filling in many methods
+ Methods for evaluating one candidate or sets of candidates
+ Pretty print methods.

<a href="gadgets.py#L486-L636"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 209:   class Gadgets:
 210:     def __init__(i,abouts):
 211:       i.abouts  = abouts
 212:   
 213:     ### Factory methods ###
 214:   
 215:     def blank(i):
 216:       "Factory for candidate objects containing Nones"
 217:       return i.abouts.clone(lambda _: None)
 218:   
 219:     def logs(i,also=None):
 220:       "Factory for candidate objects containing Logs"
 221:       new = i.abouts.clone(lambda _ : Log())
 222:       for new1,also1 in new.alongWith(also):
 223:           new1.also = also1
 224:       return new
 225:   
 226:     ##### Logging methods #####
 227:   
 228:     def log1(i,can,log):
 229:       "Stores values from 'can' into 'log'."
 230:       [log1 + x for log1,x in log.alongWith(can)]
 231:       
 232:     def logNews(i,log, news):
 233:       """Stores values from a list of cans, called 'news'
 234:          into a log. Does not use 'log1' since this
 235:          also calls the 'aggregate' method."""
 236:       for can in news:
 237:         for x,log1 in zip(can.decs,log.decs):
 238:           log1 + x
 239:         for x,log1 in zip(can.objs,log.objs):
 240:           log1 + x
 241:         log.aggregate + i.aggregate(can,log)
 242:       return news
 243:   
 244:     ##### Filling in decisions #####
 245:     
 246:     def aFewBlanks(i):
 247:       """ Handles instantiation with constraints.
 248:           If can't  make  new instance after some repeats, crash."""
 249:       patience = the.GADGETS.patience
 250:       while True:
 251:         yield i.blank()
 252:         patience -= 1
 253:         assert patience > 0, "constraints too hard to satisfy"
 254:   
 255:     def decs(i):
 256:       "return a new candidate, with guesses for decisions"
 257:       for can in i.aFewBlanks():
 258:         can.decs = [dec.maker() for dec in i.abouts.decs]
 259:         if i.abouts.ok(can):
 260:           return can
 261:   
 262:     ##### Filling in objectives #####
 263:     
 264:     def eval(i,can):
 265:       "expire the old aggregate. make the objective scores."
 266:       can.aggregate = None
 267:       can.objs = [obj.maker(can) for obj in i.abouts.objs]
 268:       return can
 269:   
 270:     def aggregate(i,can,logs):
 271:       "Return the aggregate. Side-effect: store it in the can"
 272:       if can.aggregate == None:
 273:          agg = n = 0
 274:          for obj,about,log in zip(can.objs,
 275:                                   i.abouts.objs,
 276:                                   logs.objs):
 277:            n   += 1
 278:            agg += about.fromHell(obj,log)
 279:          can.aggregate = agg ** 0.5 / n ** 0.5
 280:       return can.aggregate
 281:   
 282:     def energy(i,can,logs):
 283:       "Returns an energy value to be minimized"
 284:       i.eval(can)
 285:       e = abs(1 - i.aggregate(can,logs))
 286:       if e < 0: e= 0
 287:       if e > 1: e= 1
 288:       return e
 289:     
 290:     ##### Mutation methods #####
 291:          
 292:     def mutate(i,can,logs,p):
 293:       "Return a new can with p% mutated"
 294:       for sn in i.aFewBlanks():
 295:         for n,(dec,about,log) in enumerate(zip(can.decs,
 296:                                              i.abouts.decs,
 297:                                              logs.decs)):
 298:           val = can.decs[n]
 299:           if p > r():
 300:             some = (log.hi - log.lo)*0.5
 301:             val  = val - some + 2*some*r()
 302:             val  = about.wrap(val)
 303:           sn.decs[n] = val
 304:         if i.abouts.ok(sn):
 305:           return sn
 306:         
 307:     def xPlusFyz(i,threeMore,cr,f):
 308:       "Crossovers some decisions, by a factor of 'f'"
 309:       def smear((x1, y1, z1, about)):
 310:         x1 = x1 if cr <= r() else x1 + f*(y1-z1)
 311:         return about.wrap(x1)
 312:       for sn in i.aFewBlanks():
 313:         x,y,z   = threeMore()
 314:         sn.decs = [smear(these)
 315:                    for these in zip(x.decs,
 316:                                     y.decs,
 317:                                     z.decs,
 318:                                     i.abouts.decs)]
 319:         if i.abouts.ok(sn):
 320:           return sn
 321:   
 322:     ##### Fully filling in many candidates #####
 323:     
 324:     def news(i,n=None):
 325:       "Generating, say, 100 random instances."
 326:       return [i.eval( i.decs())
 327:               for _ in xrange(n or the.GADGETS.baseline)]
 328:   
 329:     ##### Evaluation of candidates #####
 330:     
 331:     def better1(i,now,last):
 332:       "Is one era better than another?"
 333:       better=worse=0
 334:       for now1,last1,about in zip(now.objs,
 335:                                   last.objs,
 336:                                   i.abouts.objs):
 337:         nowMed = median(now1.some())
 338:         lastMed= median(last1.some())
 339:         if about.better(nowMed, lastMed):
 340:           better += 1
 341:         elif nowMed != lastMed:
 342:           worse += 1
 343:       return better > 0 and worse < 1
 344:   
 345:     ##### Pretty printing #####
 346:       
 347:     def fyi(i,x)   :
 348:       "Maybe, mention something"
 349:       the.GADGETS.verbose and say(x)
 350:       
 351:     def shout(i,x) :
 352:       "Add an emphasis to an output."
 353:       i.fyi("__" + x)
 354:       
 355:     def bye(i,info,first,now) :
 356:       """Optimizers return the distribution of values seen in
 357:          first and final era"""
 358:       i.fyi(info)
 359:       return first,now
```

Note the last method, `bye`. What it is saying that my optimizers
return logs of what was true _before_ the optimizer ran (in the `first`
era) and _after_ the optimizer completed (in the `last` era found by the optimizer).

## Optimizers

Optimizers take (or create) some examples in some `first` era then do 
what they can to produce a new `last` era of better examples.

One detail is that, when assessing _N_ optimizers, they all have to
start at the same baseline (the same `first` era). So these optimizers
accept that baseline as an optional argument.

Note also that all the following:

+ Process a model in `era`s
+ As each era progresses, a log of what was seen is entered into `now`.
+ Between each era, `last` is set to `now` and a new log is created for
  the next `now` to be used in the next era.
+ Optimizers may stop early after a sequence of unpromising `era`s.
+ Optimizers may stop early if we get too close to zero
+ When generating logs, if there is an outer log, store values in this
  log as well as the outer (see the `also` and `also2` parameters).

### Simulated Annealing

<a href="gadgets.py#L666-L711"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 360:   @setting
 361:   def SA(): return o(
 362:       p=0.25,
 363:       cooling=1,
 364:       kmax=1000)
 365:     
 366:   def sa(m,baseline=None,also2=None):
 367:     def goodbye(x)  : return g.bye(x,first,now)
 368:     g = Gadgets(m)
 369:     def p(old,new,t): return ee**((old - new)/t)
 370:     k,eb,life = 0,1,the.GADGETS.lives
 371:     #===== setting up logs
 372:     also     = g.logs(also2) # log of all eras
 373:     first    = now  = g.logs(also)
 374:     g.logNews(first,baseline or g.news())
 375:     last, now  = now, g.logs(also)
 376:     #===== ok to go
 377:     s = g.decs()
 378:     e = g.energy(s,now)
 379:     g.fyi("%4s [%2s] %3s "% (k,life,"     "))
 380:     while True:
 381:       info="."
 382:       k += 1
 383:       t  = (k/the.SA.kmax) ** (1/the.SA.cooling)
 384:       sn = g.mutate(s, also,the.SA.p)
 385:       en = g.energy(sn,also)
 386:       g.log1(sn,now)
 387:       if en < eb:
 388:         sb,eb = sn,en
 389:         g.shout("!")
 390:       if en < e:
 391:         s,e = sn,en
 392:         info = "+"
 393:       elif p(e,en,t) < r():
 394:         s,e = sn, en
 395:         info="?"
 396:       if k % the.GADGETS.era: 
 397:         g.fyi(info)
 398:       else:
 399:         life = life - 1
 400:         if g.better1(now, last)     : life = the.GADGETS.lives 
 401:         if life < 1                 : return goodbye("L")
 402:         if eb < the.GADGETS.epsilon : return goodbye("E %.5f" %eb)
 403:         if k > the.SA.kmax          : return goodbye("K")
 404:         g.fyi("\n%4s [%2s] %.3f %s" % (k,life,eb,info))
 405:         last, now  = now, g.logs(also) 
```

### Differential Evolution

<a href="gadgets.py#L717-L760"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 406:   @setting
 407:   def DE(): return o(
 408:       cr = 0.4,
 409:       f  = 0.5,
 410:       npExpand = 10,
 411:       kmax=1000)
 412:     
 413:   def de(m,baseline=None,also2=None):
 414:     def goodbye(x)  : return g.bye(x,first,now)
 415:     g  = Gadgets(m)
 416:     np = len(g.abouts.decs) * the.DE.npExpand
 417:     k,eb,life = 0,1,the.GADGETS.lives
 418:     #===== setting up logs
 419:     also     = g.logs(also2) # also = log of all eras
 420:     first    = now  = g.logs(also)
 421:     frontier = g.logNews(first,baseline or g.news(np))
 422:     last, now  = now, g.logs(also)
 423:     #===== ok to go
 424:     sn = en = None
 425:     g.fyi("%4s [%2s] %3s "% (k,life,"     "))
 426:     while True:
 427:       for n,parent in enumerate(frontier):
 428:         info="."
 429:         k += 1
 430:         e  = g.aggregate(parent, also)
 431:         sn = g.xPlusFyz(lambda: another3(frontier,parent),
 432:                         the.DE.cr,
 433:                         the.DE.f)
 434:         en = g.energy(sn,also)
 435:         g.log1(sn,now)
 436:         if en < eb:
 437:           sb,eb = sn,en
 438:           g.shout("!")
 439:         if en < e:
 440:           frontier[n] = sn # goodbye parent
 441:           info = "+"
 442:         g.fyi(info)
 443:       life = life - 1
 444:       if g.better1(now, last)     : life = the.GADGETS.lives 
 445:       if life < 1                 : return goodbye("L")
 446:       if eb < the.GADGETS.epsilon : return goodbye("E %.5f" %eb)
 447:       if k > the.DE.kmax          : return goodbye("K")
 448:       g.fyi("\n%4s [%2s] %.3f %s" % (k,life,eb,info))
 449:       last, now  = now, g.logs(also) 
```

DE trick for finding three unique things in a list that are not `avoid`.

<a href="gadgets.py#L766-L778"><img align=right src="http://www.hungarianreference.com/i/arrow_out.gif"></a><br clear=all>
```python

 450:   def another3(lst, avoid=None):
 451:     def another1():
 452:       x = avoid
 453:       while id(x) in seen: 
 454:         x = lst[  int(random.uniform(0,len(lst))) ]
 455:       seen.append( id(x) )
 456:       return x
 457:     # -----------------------
 458:     assert len(lst) > 4
 459:     avoid = avoid or lst[0]
 460:     seen  = [ id(avoid) ]
 461:     return another1(), another1(), another1()
 462:   
```


_________

<img align=right src="https://raw.githubusercontent.com/txt/mase/master/img/pd-icon.png">Copyright © 2015 [Tim Menzies](http://menzies.us).
This is free and unencumbered software released into the public domain.   
For more details, see the [license](https://github.com/txt/mase/blob/master/LICENSE.md).


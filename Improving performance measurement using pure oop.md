# Improving performance measurement using pure object - oriented programming

This article is about how to make "**Is elem in array"** by @Анна Штыкова  more readable (more object - oriented to tell the truth).

---

First of all we must introduce interface **Measurement**, all implementations of that should be able to measure some mectic. We will use time.

```tsx
interface Measurement {
    measure() : number;
}
```

Now its time to create some class, that can measure time of function execution

```tsx
class ExecutionTime implements Measurement {
    private func : Function;
    private args : any[];

    // unfortunately, idk how to declare type of function parameter's instead of any[]
    constructor(executedFunction: Function, ...args: any[]) {
        this.func = executedFunction;
        this.args = args;
    }

    public measure() : number {
        const start = performance.now();
        this.func(this.args);
        return performance.now() - start;
    }
}
```

After that manupulations we can measure execurion time of every functions. But what if we need to know total time of some amount of executions? We have to create decorator **TotalMeasurement** that will wrap original **Measurement** (and **ExecutionTime** in particaular)

```tsx
class TotalMeasurement implements Measurement {
    private measurement : Measurement;
    private launches : number;

    constructor(measurement : Measurement, launches : number) {
        if (launches <= 0) {
            throw new Error('"launches" must be positive inreger');
        }
        this.measurement = measurement;
        this.launches = launches;
    }

    public measure() : number {
        let totalMeasure : number = 0.0;
        for (let launch = 0; launch < this.launches; launch++) {
            totalMeasure += this.measurement.measure();
        }
        return totalMeasure;
    }
}
```

Now we can measure all we need. 

---

Firstly, let's measure time of single call.

```tsx
let array : Array<number> = new Array<number>(10_000); 
for (let i = 0; i < 10_000; i++) {
    array[i] = i + 1;
}

// we need this variable to measure only time of search in set
let setFromArray = new Set<number>(array);
let setCreationTime = new ExecutionTime(() => new Set<number>(array)).measure();

let executionTimes = {
    'arrayIndexOf'  : new ExecutionTime(() => array.indexOf(10_000)),
    'setFound'      : new ExecutionTime(() => setFromArray.has(10_000)) 
};

for (let [name, time] of Object.entries(executionTimes)) {
    console.log(`Time of ${name} is ${time.measure()}`);
}
```

We will get such results:

> Time of arrayIndexOf is 0.0350000336766243
Time of setFound is 0.034999975468963385

---

Let's find out how much faster an **indexOf** that a **set** realization:

```tsx
let setSingleTime = setCreationTime + executionTimes.setFound.measure();

console.log(`IndexOf is `${executionTimes.arrayIndexOf.measure() / setSingleTime} times faster that set realization`);
```

> IndexOf is 44.999918509420894 times faster that set realization

As we can see, if the number of searches is small, then **IndexOf** is much faster.

---

Now it remains to answer the last question: what if we have to find value more than one time? For example n/2 times in array of length n?

```tsx
const launches = array.length / 2;

const totalExecutionTimes = {
    'arrayIndexOf' : new TotalMeasurement(executionTimes.arrayIndexOf, launches),
    'setFound'     : new TotalMeasurement(executionTimes.setFound, launches) 
};

const setTotalTime = setCreationTime + totalExecutionTimes.setFound.measure();

console.log(`IndexOf is ${executionTimes.arrayIndexOf.measure() / setTotalTime} 
            times slower that set realization`);
```

> IndexOf is 11.351515120676316 times slower that set realization

It means that we always have to estimate the number of searches to decide which method to use.

---

### Conslusions:

1. Introduced **interface Measurement** that can be reused later for every types of measure's
2. Introduced **class TotalMeasurement** that can be reused to measure lot of "measure" call's
3. Introduced **class ExecutionTime** that will help us to measure time of function execution
4. We found out that when amount of searches in array is about n/k for some positive integer k, it's more reasonable to convert it to set.

---

                                                                            Author: @Sergey Kashtanov 

                                                                            Date: May 18, 2021

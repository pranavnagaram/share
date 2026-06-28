# **Getting Started with BRAIN Python Alphas**

**Python Alpha** is a new feature that enables you to write Alphas in Python. By using Python’s rich open-source ecosystem, you can build Alphas that are more granular, flexible, and diverse than ever before.

You can access this feature through the Platform UI by [selecting Python as the language](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#option-d-platform-uiapi-simulation), or conduct research in the [BrainLabs environment](https://platform.worldquantbrain.com/profile/account/brainlabs).

## **Table of Contents**

* [Setup](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#setup)  
  * [Handling integer fields](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#handling-integer-fields)  
* [The @alpha Decorator](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#the-alpha-decorator)  
  * [data — Declaring Input Fields](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#data--declaring-input-fields)  
  * [store — Persisting State Across Time Steps](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#store--persisting-state-across-time-steps)  
  * [Return Value](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#return-value)  
  * [Rules Summary](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#rules-summary)  
* [Adjusting Prices for Corporate Actions](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#adjusting-prices-for-corporate-actions)  
  * [Example: stock split on 2014-12-22](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#example-stock-split-on-2014-12-22)  
* [Two Simulation Paths](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#two-simulation-paths)  
  * [Path 1: BrainLabs Simulation (fast, local)](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#path-1-brainlabs-simulation-fast-local)  
  * [Path 2: Actual Simulation (accurate, remote)](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#path-2-actual-simulation-accurate-remote)  
  * [When to Use Which](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#when-to-use-which)  
* [Simulating an Alpha](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#simulating-an-alpha)  
  * [Option A: Send the decorated function directly](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#option-a-send-the-decorated-function-directly)  
  * [Option B: Read the code file and send it](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#option-b-read-the-code-file-and-send-it)  
  * [Option C: Async simulation](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#option-c-async-simulation)  
  * [Option D: Platform UI/API Simulation](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#option-d-platform-uiapi-simulation)  
* [SimulationSettings Reference](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#simulationsettings-reference)  
* [Complete Example](https://platform.worldquantbrain.com/learn/documentation/consultant-information/getting-started-brain-python-alphas#complete-example)

---

## **Setup**

**from** **brain** **import** Brain, BrainCache  
**from** **brain.models** **import** SimulationSettings  
**from** **brain.alphas** **import** alpha  
**import** **numpy** **as** **np**  
**import** **numpy.typing** **as** **npt**

brain \= Brain(raw\=**True**)  
cache \= BrainCache(brain)

Brain() defaults to instrument\_type='EQUITY', region='USA', delay=1, universe='TOP3000'. Pass raw=True to receive data in its native dtype (e.g., float32, int32) — the same types used in the actual simulation environment. Without raw=True, all data is converted to float64.

BrainCache wraps Brain and caches data field lookups — pass it to every simulation call to avoid redundant downloads. Create a new BrainCache if you change the simulation region, delay, or date range, since cached data is tied to those parameters. BrainCache is reserved for simulation only — to explore or inspect data, use brain.get\_data\_frame() directly:

*\# Exploring data — use brain.get\_data\_frame(), NOT cache*  
df \= brain.get\_data\_frame("returns")  
print(df.shape)    *\# (n\_dates, n\_instruments)*  
print(df.dtypes)

### **Handling integer fields**

Some data fields are integer-typed (e.g., int32). Integer arrays cannot hold NaN — missing values are represented by the type's minimum value (np.iinfo(np.int32).min, i.e., \-2147483648). To work with these fields, cast to float32 and replace the sentinel with NaN:

**from** **brain** **import** get\_missing\_value

data \= brain.get\_data\_frame('pv96\_acquisition\_c\_flag').values  
print(data.dtype)   *\# int32*

missing \= get\_missing\_value(data.dtype)   *\# np.iinfo(np.int32).min*  
data\_float \= data.astype(np.float32)  
data\_float\[data \== missing\] \= np.nan

---

## **The @alpha Decorator**

Every Alpha is a Python function decorated with @alpha. The decorator declares which data fields to load and which store variables to persist across time steps.

**@alpha**(  
    data\=\["returns"\],  
    store\=\["my\_state"\],  
)  
**def** my\_alpha(data, store) \-\> npt.NDArray\[np.float32\]:  
    *\# data.returns — shape \[lookback\_window, n\_instruments\], float32*  
    *\# data.universe — shape \[lookback\_window, n\_instruments\] (always available, int: 1 \= in, 0 \= out)*  
    *\#*  
    *\# store.my\_state — initialized to None, persists across time steps*

    signal \= \-np.nanmean(data.returns, axis\=0)  
    **return** signal.astype(np.float32)

### **data — Declaring Input Fields**

The data parameter is a list of data field names. Each name becomes an attribute on the data namespace as a 2-D numpy array shaped \[lookback\_window, n\_instruments\]:

**@alpha**(data\=\["returns", "close"\], store\=\[\])  
**def** my\_alpha(data, store) \-\> npt.NDArray\[np.float32\]:  
    *\# data.returns — \[lookback\_window, n\_instruments\], float32*  
    *\# data.close — \[lookback\_window, n\_instruments\], float32*  
    *\# data.universe — \[lookback\_window, n\_instruments\] (always available, do NOT list it)*  
    ...

The simulation engine calls your Alpha once per time step. At each step, each data field contains lookback \+ 1 rows of history (set via SimulationSettings.lookback). A lookback of 0 gives 1 row (today only), a lookback of 5 gives 6 rows. The most recent data is the last row:

yesterday \= data.returns\[\-1\]          *\# shape: \[n\_instruments\]*  
two\_days\_ago \= data.returns\[\-2\]       *\# shape: \[n\_instruments\]*  
full\_window \= data.returns            *\# shape: \[lookback\_window, n\_instruments\]*

During the first few time steps the window is still growing (fewer than lookback \+ 1 rows). Once the window reaches full size it slides forward — one new row enters at the bottom, the oldest drops off the top.

The special field universe is always available as an integer array (1 \= in-universe, 0 \= out). Do not include "universe" in your data list.

**Data arrays are read-only.** The engine marks them as non-writable — attempting to modify them in-place will raise a ValueError. Always .copy() before modifying:

*\# WRONG — raises ValueError: assignment destination is read-only*  
a \= data.returns\[\-1\]  
a\[a \< 0\] \= 0

*\# RIGHT — copy first*  
a \= data.returns\[\-1\].copy()  
a\[a \< 0\] \= 0

### **store — Persisting State Across Time Steps**

The store parameter is a list of store declarations. Each is either a plain **string** (untyped, pre-initialized to None) or a **typed dict** that declares the variable's shape and how it grows with the universe:

{"name": "my\_arr", "dims": "i", "extend": np.float64(0)}

* "name": variable name (required)  
* "dims": string of axis characters — "i" \= instruments axis (auto-extended when universe grows), "x" \= free axis. "xi" → 2D \[any, n\_instruments\].  
* "extend": fill value for new instruments — **must match the array's dtype exactly**. Always specify it explicitly, even when the default may suffice — it makes intent clear and avoids surprises. Use explicit NumPy scalar constructors: np.float64(0), np.float32(0), np.float64(np.nan) for float64, np.float32(np.nan) for float32. Never use bare np.nan — it is Python's float, not numpy.float64, and will fail the type check. Never use bare Python literals like 0 or 0.0 — they may not match the array's dtype and will raise a ValueError: Store item 'name' extend value '0' has wrong type: \<class 'int'\> expected: \<class 'numpy.float32'\> If omitted, the fill is type-determined (NaN for float arrays, the type's minimum value for integers), but omitting is discouraged.

Use store.var is None to detect the first call. For typed entries the simulator auto-extends "i" axes when the universe grows — no manual padding needed.

**@alpha**(data\=\["returns"\], store\=\[{"name": "running\_mean", "dims": "i", "extend": np.float64(0)}\])  
**def** smooth\_alpha(data, store) \-\> npt.NDArray\[np.float32\]:  
    **if** store.running\_mean **is** **None**:  
        store.running\_mean \= np.zeros(data.returns.shape\[1\], dtype\=np.float64)

    raw \= \-np.nanmean(data.returns, axis\=0)   *\# float64*  
    store.running\_mean \= 0.9 \* store.running\_mean \+ 0.1 \* raw  
    **return** store.running\_mean.astype(np.float32)

Use "dims": "xi" for a 2D cache where axis 0 is a free (time) axis and axis 1 is the instruments axis. A practical use case is caching cross-sectional ranks across more days than the lookback window provides. Use "extend": np.float64(np.nan) so historical entries for newly added instruments are NaN — np.nanmean then naturally excludes them until the instrument accumulates history:

RANK\_DAYS \= 20

**@alpha**(  
    data\=\["returns"\],  
    store\=\[  
        {"name": "rank\_cache", "dims": "xi", "extend": np.float64(np.nan)},  *\# \[n\_days\_cached, n\_instruments\]*  
    \],  
)  
**def** mean\_of\_rank\_alpha(data, store) \-\> npt.NDArray\[np.float32\]:  
    today \= data.returns\[\-1\]

    *\# Cross-sectional rank of today's return*  
    finite \= np.where(np.isnan(today), \-np.inf, today)  
    today\_rank \= np.argsort(np.argsort(finite)).astype(np.float64)  
    today\_rank\[np.isnan(today)\] \= np.nan

    **if** store.rank\_cache **is** **None**:  
        store.rank\_cache \= today\_rank\[np.newaxis, :\]  *\# shape: \[1, n\_instruments\]*  
    **else**:  
        new\_cache \= np.vstack(\[store.rank\_cache, today\_rank\[np.newaxis, :\]\])  
        store.rank\_cache \= new\_cache\[\-RANK\_DAYS:\]

    mean\_rank \= np.nanmean(store.rank\_cache, axis\=0)  
    **return** (\-mean\_rank).astype(np.float32)

When the universe grows, the simulator appends a new NaN column to rank\_cache — no manual padding needed.

Valid values for untyped string entries: \- None, scalars (int, float, bool, str), NumPy scalars \- numpy.ndarray \- Lists and dicts containing any of the above, including nested arrays, lists, and dicts \- Dict keys must be strings

**Fallback — \_pad\_store for untyped entries**: if you use a plain string entry that stores an instrument-sized array, extend it manually before reading otherwise it will throw error in mid-simulation:

**def** \_pad\_store(arr, n\_insts, fill\_value):  
*"""Pad along the instruments axis when the universe grows."""*  
    **if** arr.shape\[\-1\] \>= n\_insts:  
        **return** arr  
    n\_new \= n\_insts \- arr.shape\[\-1\]  
    **if** arr.ndim \== 1:  
        **return** np.pad(arr, (0, n\_new), mode\='constant', constant\_values\=fill\_value)  
    **return** np.pad(arr, ((0, 0), (0, n\_new)), mode\='constant', constant\_values\=fill\_value)

n\_insts \= data.returns.shape\[1\]  
store.running\_sum \= \_pad\_store(store.running\_sum, n\_insts, fill\_value\=0.0)

But even 2D cases like a correlation matrix (diagonal=1, off-diagonal=0) work with typed dicts: use "dims": "ii" with "extend": np.float64(0). The simulator fills new rows and columns with 0; fix the diagonal for newly added instruments in the Alpha body using a plain string "prev\_n" to track the previous universe size.

### **Return Value**

The Alpha must return a 1-D float32 numpy array of shape \[n\_instruments\]. Data fields from BRAIN are typically float32, but many NumPy operations silently promote to float64 — np.nanmean, np.nanstd, np.nansum, and even arithmetic with Python floats like 0.9 \* array. Always add .astype(np.float32) at the return as a safety measure (it is a no-op if the array is already float32):

**def** my\_alpha(data, store) \-\> npt.NDArray\[np.float32\]:  
    *\# np.nanmean promotes float32 input to float64 output*  
    signal \= \-np.nanmean(data.returns, axis\=0)  
    **return** signal.astype(np.float32)

If the return value is not float32, the simulation will raise: Alpha vector is not float32.

### **Rules Summary**

1. Exactly one @alpha decorator per function  
2. Function must accept exactly 2 parameters (data, store)  
3. Return type must be float32 numpy array of shape \[n\_instruments\]  
4. Do not include "universe" in the data list — it is always available  
5. Do not mutate data arrays in-place — they are read-only, copy first

---

## **Adjusting Prices for Corporate Actions**

Any per-share field contains discontinuities from corporate actions such as stock splits. This includes prices (close, open, high, low) as well as per-share fundamentals like eps, book\_value\_ps, and similar. For example, a 1:5 reverse split causes the reported price to jump overnight — not a real market move. If you use these fields directly in Alpha logic, the jumps will produce false signals.

Brain provides an adjfactor field that records cumulative adjustment factors for each instrument. Apply it to convert raw prices into a continuous, split-adjusted series:

**import** **matplotlib.pyplot** **as** **plt**

close \= brain.get\_data\_frame('close')  
adjfactor \= brain.get\_data\_frame('adjfactor')

*\# Compute split-adjusted close prices*  
adjusted\_close \= close / ((adjfactor \- 1.0).cumsum() \+ 1.0)

### **Example: stock split on 2014-12-22**

Instrument EQ0000000000041095 had a 1:5 reverse split on 2014-12-22. Without adjustment, the price appears to jump from \~0.5 to \~3.5:

instrument \= 'EQ0000000000041095'  
date\_range \= slice('2014-12-01', '2015-01-31')

fig, axes \= plt.subplots(1, 2, figsize\=(14, 5))

close.loc\[date\_range, instrument\].plot(ax\=axes\[0\], title\='Raw close price')  
axes\[0\].set\_ylabel('Price')

adjusted\_close.loc\[date\_range, instrument\].plot(ax\=axes\[1\], title\='Split-adjusted close price')  
axes\[1\].set\_ylabel('Price')

plt.tight\_layout()  
plt.show()

The raw chart shows a sharp discontinuity at the split date. The adjusted chart shows a smooth, continuous price series — which is what your Alpha should operate on.

**Tip**: The returns field is already split-adjusted. This adjustment step is needed for any per-share field — prices (close, open, high, low), per-share fundamentals (eps, book\_value\_ps), and similar.

---

## **Two Simulation Paths**

### **Path 1: BrainLabs Simulation (fast, local)**

BrainLabs simulation runs your Alpha function locally on cached data. It executes row-by-row through a three-step pipeline. This is fast and ideal for development, debugging, and parameter sweeps — but the PnL may differ slightly from the actual simulation environment.

simulation\_settings \= SimulationSettings(  
    instrument\_type\='EQUITY',  
    region\='USA',  
    delay\=1,  
    universe\='TOP3000',  
    lookback\=5,  
    visualization\=**True**,  
)

*\# Step 1: Generate alpha matrix (calls your alpha function row-by-row)*  
alpha\_matrix \= brain.generate\_alpha\_matrix(mean\_reversion, simulation\_settings, cache)

*\# Step 2: Generate positions (applies delay, pasteurization, neutralization, scaling)*  
positions \= brain.generate\_alpha\_positions(alpha\_matrix, simulation\_settings, cache)

*\# Step 3: Generate stats (computes daily PnL, prints Sharpe/returns/drawdown/turnover)*  
stats \= brain.generate\_alpha\_stats(positions, simulation\_settings, cache)

### **Path 2: Actual Simulation (accurate, remote)**

Actual simulation submits your Alpha to the BRAIN backend. This produces accurate results but is slower (remote API call with polling). The backend applies the full pipeline including decay, neutralization, pasteurization, truncation, and position/per-step position change limits.

simulation\_settings \= SimulationSettings(  
    instrument\_type\='EQUITY',  
    region\='USA',  
    delay\=1,  
    universe\='TOP3000',  
    lookback\=5,  
    decay\=10,  
    neutralization\='MARKET',  
    pasteurization\='ON',  
    truncation\=0.8,  
    language\='PYTHON',  
    visualization\=**False**,  
    max\_position\='OFF',  
    max\_trade\='OFF',  
)

*\# Submit the @alpha-decorated function*  
alpha\_id \= brain.simulate(mean\_reversion, simulation\_settings)  
print(alpha\_id)

*\# Retrieve the result*  
alpha\_result \= brain.get\_alpha(alpha\_id)  
print(alpha\_result)

### **When to Use Which**

**Speed**  
Fast (local, cached data)  
Slower (remote API)  
**Accuracy**  
Approximate  
Accurate  
**Use for**  
Development, debugging, parameter sweeps  
Final validation, submission  
**Pipeline**  
generate\_alpha\_matrix \-\> generate\_alpha\_positions \-\> generate\_alpha\_stats  
brain.simulate(alpha\_fn, settings)  
**Recommended workflow**: Develop and iterate using BrainLabs simulation (fast feedback), then validate your best Alpha with actual simulation before submission.

---

## **Simulating an Alpha**

### **Option A: Send the decorated function directly**

Pass your @alpha\-decorated function to brain.simulate():

alpha\_id \= brain.simulate(mean\_reversion, simulation\_settings)

The function source code is extracted automatically and sent to the backend. All helper functions and imports used inside the Alpha must be visible in the same notebook cell.

### **Option B: Read the code file and send it**

Save your Alpha and all its dependencies into a single .py file, then read and submit it as a string:

*\# alpha\_example.py*  
**from** **brain.alphas** **import** alpha  
**import** **numpy** **as** **np**  
**import** **numpy.typing** **as** **npt**

**def** pasteurize(a: npt.NDArray\[np.float32\], u: npt.NDArray) \-\> npt.NDArray\[np.float32\]:  
    *\# return pasteurized position, removing out-of-universe position*

**def** neutralize\_scale(a: npt.NDArray\[np.float32\]) \-\> npt.NDArray\[np.float32\]:  
    *\# return neutralized and scaled position*

**@alpha**(  
    data\=\["returns"\],  
    store\=\[\],  
)  
**def** mean\_reversion(data, store) \-\> npt.NDArray\[np.float32\]:  
    a \= \-np.nanmean(data.returns, axis\=0).astype(np.float32)  
    a \= pasteurize(a, data.universe\[\-1\])  
    a \= neutralize\_scale(a)  
    **return** a.astype(np.float32)

Then submit from your notebook:

brain \= Brain(raw\=**True**)

simulation\_settings \= SimulationSettings(  
    instrument\_type\='EQUITY', region\='USA', delay\=1,  
    universe\='TOP3000', lookback\=5, decay\=10,  
    neutralization\='MARKET', pasteurization\='ON',  
    truncation\=0.8, language\='PYTHON',  
    visualization\=**False**, max\_position\='OFF', max\_trade\='OFF',  
)

**with** open('./alpha\_example.py', 'r') **as** f:  
    code \= f.read()

alpha\_id \= brain.simulate(code, simulation\_settings)  
print(alpha\_id)

This is useful when your Alpha has helper functions or operators that need to travel with the submission as a single unit.

### **Option C: Async simulation**

For non-blocking submission — useful when running multiple simulations concurrently or from an async context — use the async API:

**import** **asyncio**

simulation\_id \= **await** brain.simulate\_async(code, simulation\_settings)  
print('started', simulation\_id)  
simulation\_response, retry\_after, alpha\_id \= **await** brain.get\_simulation\_async(simulation\_id)  
**while** retry\_after:  
    print('sleeping', retry\_after)  
    **await** asyncio.sleep(retry\_after)  
    simulation\_response, retry\_after, alpha\_id \= **await** brain.get\_simulation\_async(simulation\_id)

print('completed', simulation\_id, alpha\_id, simulation\_response)

simulate\_async accepts either a @alpha\-decorated function or a code string, same as brain.simulate. get\_simulation\_async returns a (response, retry\_after, alpha\_id) tuple — retry\_after is non-zero while the simulation is still running, and None once complete.

### **Option D: Platform UI/API Simulation**

Python Alpha can be simulated and submitted on BRAIN just like Fast Expression Alpha. Simply open the LANGUAGE dropdown in Settings, select Python, and enter your Python code in the code editor below. You may notice that the parameters in Settings are slightly different from those used for Fast Expression.

![Platform Simulation][image1]

Once the Python code is ready, simply click the Simulate button to run the simulation. You will see a progress bar during execution, and the results will be displayed when the simulation is complete. If you encounter a failure message, consider first running your Python Alpha through the Path 1 BrainLabs environment to further debug your code.

Remember not to import packages that are not used directly in the Alpha function. For example, you should remove from brain import Brain, BrainCache and from brain.models import SimulationSettings when simulating through UI.

---

## **SimulationSettings Reference**

instrument\_type  
Market type  
'EQUITY'  
region  
Simulation region  
'USA', 'EUR', 'ASI'  
delay  
Delay  
0, 1  
universe  
Instrument universe  
'TOP3000', 'TOP2000', 'TOP500'  
lookback  
Extra history rows (window \= lookback \+ 1\)  
0, 5, 21, 63, etc.  
start\_date  
Simulation start date (BrainLabs sim)  
'2020-01-01'  
end\_date  
Simulation end date (BrainLabs sim)  
'2021-12-31'  
neutralization  
Position neutralization  
'NONE', 'MARKET', 'SECTOR'  
truncation  
Position limit factor  
0.01, 0.05  
decay  
Alpha decay parameter  
0, 3, 6, 10  
pasteurization  
Mask to universe  
'ON', 'OFF'  
language  
Simulation language (actual sim)  
'PYTHON'  
visualization  
Show PnL chart (BrainLabs sim)  
True, False  
max\_position  
Limit position sizes (actual sim)  
'ON', 'OFF'  
max\_trade  
Limit per-step position changes (actual sim)  
'ON', 'OFF'  
---

## **Complete Example**

The example below is meant to be run in a notebook in **BrainLabs**. Each fenced block is a separate cell — run them from top-to-bottom.

### **BrainLabs simulation (fast iteration)**

**from** **brain** **import** Brain, BrainCache  
**from** **brain.models** **import** SimulationSettings  
**from** **brain.alphas** **import** alpha  
**import** **numpy** **as** **np**  
**import** **numpy.typing** **as** **npt**

**def** pasteurize(a, u):  
    a \= a.copy()  
    a\[\~u.astype(bool)\] \= np.nan  
    **return** a

**def** neutralize(a):  
    a0 \= np.nan\_to\_num(a, nan\=0, posinf\=0, neginf\=0)  
    **return** a \- np.mean(a0)

**def** scale(a):  
    a0 \= np.nan\_to\_num(a, nan\=0, posinf\=0, neginf\=0)  
    norm \= np.linalg.norm(a0, ord\=1)  
    **return** a / norm **if** norm \> 0 **else** a

**@alpha**(  
    data\=\["returns"\],  
    store\=\[\],  
)  
**def** mean\_reversion(data, store) \-\> npt.NDArray\[np.float32\]:  
    a \= \-np.nanmean(data.returns, axis\=0).astype(np.float32)  
    a \= pasteurize(a, data.universe\[\-1\])  
    a \= scale(neutralize(a))  
    **return** a.astype(np.float32)

brain \= Brain(raw\=**True**)  
cache \= BrainCache(brain)

settings \= SimulationSettings(  
    instrument\_type\='EQUITY', region\='USA', delay\=1,  
    universe\='TOP3000', lookback\=5, visualization\=**True**,  
)

alpha\_matrix \= brain.generate\_alpha\_matrix(mean\_reversion, settings, cache)  
positions \= brain.generate\_alpha\_positions(alpha\_matrix, settings, cache)  
stats \= brain.generate\_alpha\_stats(positions, settings, cache)

### **Actual simulation (final validation \+ submission)**

**Note**: brain.simulate(...) reads the **entire source cell** that defines the Alpha function and ships it to the simulation server. Only importing alpha module from brain.alphas is allowed in the **same cell** as the @alpha\-decorated function. Importing other brain modules such as from brain import Brain, BrainCache or from brain.models import SimulationSettings, the simulation will fail. Keep those imports out of the Alpha cell. The same import restriction applies when simulating Python Alpha from BRAIN.

**Cell 1 — Alpha definition** (only from brain.alphas import alpha is put here)

**from** **brain.alphas** **import** alpha  
**import** **numpy** **as** **np**  
**import** **numpy.typing** **as** **npt**

**def** pasteurize(a, u):  
    a \= a.copy()  
    a\[\~u.astype(bool)\] \= np.nan  
    **return** a

**def** neutralize(a):  
    a0 \= np.nan\_to\_num(a, nan\=0, posinf\=0, neginf\=0)  
    **return** a \- np.mean(a0)

**def** scale(a):  
    a0 \= np.nan\_to\_num(a, nan\=0, posinf\=0, neginf\=0)  
    norm \= np.linalg.norm(a0, ord\=1)  
    **return** a / norm **if** norm \> 0 **else** a

**@alpha**(  
    data\=\["returns"\],  
    store\=\[\],  
)  
**def** mean\_reversion(data, store) \-\> npt.NDArray\[np.float32\]:  
    a \= \-np.nanmean(data.returns, axis\=0).astype(np.float32)  
    a \= pasteurize(a, data.universe\[\-1\])  
    a \= scale(neutralize(a))  
    **return** a.astype(np.float32)

**Cell 2 — submit and fetch the result** (Brain / SimulationSettings imports go here)

**from** **brain** **import** Brain  
**from** **brain.models** **import** SimulationSettings

brain \= Brain(raw\=**True**)

submit\_settings \= SimulationSettings(  
    instrument\_type\='EQUITY', region\='USA', delay\=1,  
    universe\='TOP3000', lookback\=5, decay\=10,  
    neutralization\='MARKET', pasteurization\='ON',  
    truncation\=0.8, language\='PYTHON',  
    visualization\=**False**, max\_position\='OFF', max\_trade\='OFF',  
)

alpha\_id \= brain.simulate(mean\_reversion, submit\_settings)  
alpha\_result \= brain.get\_alpha(alpha\_id)  
print(alpha\_result)  


[image1]: <data:image/png;base64,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>
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

How to translate Fast Expression Alpha into Python Alpha
Translating an existing Fast Expression Alpha into Python is a great way to write your first Python Alpha. It is not only easier than starting from scratch, but also lets you directly compare performance between the Python version and the original — giving you a concrete baseline to build on as you think about ways to improve it by leveraging the full flexibility of the Python language.

This article has two parts:

How to prompt an LLM to translate a Fast Expression Alpha into a Python Alpha
Tips and techniques for making Python Alphas better than the underlying expression
Part 1 — Prompting an LLM to Translate a Fast Expression Alpha
We encourage to prompt the LLM with at least following five sections for converting fast expression into python properly

Original Fast Expression
Operator description
Data field description
Python Alpha syntax
Simple example
1.1 — The Original Fast Expression
For example, your original Alpha expression:

ts_rank(rank(returns))
1.2 — Description of the Operators Used
You can find descriptions of all operators on the operator page.

ts_rank — The ts_rank operator evaluates how the current value of a variable compares to its values over a defined lookback period (d days) for each instrument. It returns a normalized rank (between 0 and 1) of the current value within that window, optionally shifted by a constant. This is helpful for identifying trends, momentum, or reversals in time-series data.

rank — The rank(x) operator assigns a rank to each value in the input x across all instruments for a given date, mapping the lowest value to 0.0 and the highest to 1.0, with all other values evenly distributed in between. This helps normalize data, limit extreme values, and can improve the stability of your Alpha by reducing outliers and drawdown. The optional rate parameter controls the precision of sorting (default is 2; set to 0 for exact sorting).

1.3 — Description of the Data Fields Used
You can find data field descriptions on the data explorer page.

returns — Daily returns

1.4 — The Python Alpha Syntax
The @alpha contract

Every Python Alpha is one self-contained module. It cannot import your local files, so every helper function must be contained in the file itself.

from brain.alphas import alpha
import numpy as np
import numpy.typing as npt

@alpha(
    data=["field_a", "field_b"],   # MATRIX fields to load; do NOT list "universe"
    store=[],                       # state persisted across days (often empty)
)
def my_alpha(data, store) -> npt.NDArray[np.float32]:
    ...
    return signal.astype(np.float32)
Rules that bite:

The engine calls the function once per time step. Each data.<field> is a 2-D array [rows, n_instruments], dtype float32, where rows = lookback+1 when the window is full and today is the last row (field[-1]).
During warm-up the window has fewer rows (can be 1). Reductions over [-d:] are safe; explicit back-indexing (x[-1-d]) must guard x.shape[0].
data.universe is always present (int 1/0). Never put "universe" in data. Use data.universe[-1] for today's in-universe mask.
Input data field arrays are read-only. .copy() before mutating, or you get "assignment destination is read-only."
Return a 1-D float32 array of shape (n_instruments,). numpy silently promotes to float64 (np.nanmean, multiplying by a Python float, …), so always finish with .astype(np.float32) — otherwise "Alpha vector is not float32."
Only MATRIX fields load. VECTOR fields, the GLB region, and multi-sim are not supported.
Set the simulation lookback ≥ the largest ts_* window used. Sparse fundamentals want lookback ≈ 250 so backfill has room.
store — state across days. Most ports need none. Use it only for path-dependent operators (trade_when, keep, hump, days_from_last_change, running means, buffers). A typed entry auto-extends along the instrument axis as the universe grows:

store=[{"name": "buf", "dims": "i", "extend": np.float64(0)}]   # "i"=1-D vector, "xi"=2-D [any, n_insts]
Detect the first call with store.<name> is None.

Full store + extend example — rolling window buffer with "dims": "xi"

This example keeps a rolling 10-day window of raw returns per instrument in a 2-D store [window, n_instruments] and outputs the mean-reversion of that window. "dims": "xi" means the x axis is free (the 10-day window) and the i axis is along instruments — the simulator auto-extends new instrument columns with the extend fill value.

from brain.alphas import alpha
import numpy as np
import numpy.typing as npt

WINDOW = 10

@alpha(
    data=["returns"],
    store=[{"name": "buf", "dims": "xi", "extend": np.float64(0)}],
)
def rolling_reversion(data, store) -> npt.NDArray[np.float32]:
    raw = -np.nanmean(data.returns, axis=0)          # float64, shape [n_instruments]

    if store.buf is None:
        # First call: initialise buffer with this single row
        store.buf = raw[np.newaxis, :]               # shape [1, n_instruments]
    else:
        # Append today's row and keep only the last WINDOW rows
        store.buf = np.vstack([store.buf, raw])[-WINDOW:]   # shape [≤WINDOW, n_instruments]

    signal = np.nanmean(store.buf, axis=0)           # float64, shape [n_instruments]
    return signal.astype(np.float32)
Key points:

"dims": "xi" — the x axis (rows = window length) is free/fixed by your logic; the i axis is instruments. When a new instrument enters the universe the simulator appends a new column filled with extend (0.0).
"dims": "i" would be for a plain 1-D vector; use "xi" whenever you need a 2-D buffer shaped [any_rows, n_instruments].
"extend": np.float64(0) must match the array's dtype exactly — np.float32(0) for float32 stores, np.int64(0) for int64, etc.
store.buf is None is only true on the very first call; after that it is always a numpy array.
Assign back to store.buf every step — the simulator persists whatever you leave in store between calls.
1.5 — A Python Alpha Example
from brain.alphas import alpha
import numpy as np
import numpy.typing as npt


def pasteurize(a, u):
    a = a.copy()
    a[~u.astype(bool)] = np.nan
    return a

def neutralize(a):
    a0 = np.nan_to_num(a, nan=0, posinf=0, neginf=0)
    return a - np.mean(a0)

def scale(a):
    a0 = np.nan_to_num(a, nan=0, posinf=0, neginf=0)
    norm = np.linalg.norm(a0, ord=1)
    return a / norm if norm > 0 else a


@alpha(
    data=["returns"],
    store=[],
)
def mean_reversion(data, store) -> npt.NDArray[np.float32]:
    a = -np.nanmean(data.returns, axis=0).astype(np.float32)
    a = pasteurize(a, data.universe[-1])
    a = scale(neutralize(a))
    return a.astype(np.float32)
You can also, for easier reuse, turn the data fields and operator descriptions into a lookup via MCP using the BRAIN API, and then save the entire prompt construction as a skill. That way, next time the LLM agent can directly use that skill to convert a Fast Expression Alpha into a Python Alpha.

For tracking the record, please tag translated Alphas with original fast expression Alpha ID. There is an ID column in Alphas page.

Part 2 — Tips, Tricks, and Techniques for Better Python Alphas
Better data preprocessing — clean and normalize inputs more carefully than the Fast Expression engine does by default.
Use machine learning algorithms — capture relationships between instruments reflected in the data itself. For example, use various clustering algorithms to create customized groupings to enhance Alpha performance.
Use advanced mathematical models — apply techniques like Fourier Transformation to find the essential signal driving Alpha performance and decouple it from noise.
Use heuristic optimization — apply methods such as ant-colony optimization (ACO) to search for better Alpha parameters in a time-efficient way.
Research note: While Python Alphas make it easy to increase model complexity, the fundamental principles of Alpha research still apply. Keep the core idea simple, avoid overfitting, and do not add complexity unless it delivers a meaningful improvement. If a technique as involved as deep learning only moves Sharpe from 2.0 to 2.05, the marginal gain does not justify the added overfitting risk — and the Alpha is likely worse off for it.
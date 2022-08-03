```python
import numpy as np
import matplotlib.pyplot as plt
```


```python
def plot_function(f,tmin,tmax,tlabel=None,xlabel=None,axes=False, **kwargs):
    ts = np.linspace(tmin,tmax,1000)
    if tlabel:
        plt.xlabel(tlabel,fontsize=18)
    if xlabel:
        plt.ylabel(xlabel,fontsize=18)
    plt.plot(ts, [f(t) for t in ts], **kwargs)
    if axes:
        total_t = tmax-tmin
        plt.plot([tmin-total_t/10,tmax+total_t/10],[0,0],c='k',linewidth=1)
        plt.xlim(tmin-total_t/10,tmax+total_t/10)
        xmin, xmax = plt.ylim()
        plt.plot([0,0],[xmin,xmax],c='k',linewidth=1)
        plt.ylim(xmin,xmax)
```


```python
def plot_volume(f,tmin,tmax,axes=False,**kwargs):
    plot_function(f,tmin,tmax,axes=axes, **kwargs)

def plot_flow_rate(f,tmin,tmax,axes=False,**kwargs):
    plot_function(f,tmin,tmax,axes=axes, **kwargs)
```


```python
def volume(t):
    return 5*(t**2)+t+5

def volume2(t):
    return 2*(t**2)-t+2

def total_function(t):
    return 7*(t**2)+7

def volume3(t):
    return -7*(t**2)+7
```


```python
plot_volume(volume,-1,1)
plot_volume(volume3,-1,1)
```


    
![output_4_0](https://user-images.githubusercontent.com/100830660/182598514-353df6c6-ef57-4995-93db-b4ac42cb3935.png)
    



```python
plot_volume(total_function,-1,1,color='red')
plot_volume(volume2,-1,1,color='orange')
plot_volume(volume,-1,1)
```


    
![output_5_0](https://user-images.githubusercontent.com/100830660/182598520-31344a68-5336-49e6-9dd3-ec5c8e688f5e.png)
    


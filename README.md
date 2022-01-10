# QuantConnect Code Snippets

Examples and references, meant for copying and pasting as needed. https://quantconnect.com

## Universe Selection
One of the challenges I had is how to control frequency of Universe selection. This [forum post](https://www.quantconnect.com/forum/discussion/4623/documentation-discussion-algorithm-reference-universes/p1/comment-16957) list 3 alternatives to address that, using ScheduledUniverseSelection, data consolidator and flags. Using flags allows using Course and Fine filters easily, as per code below.

### Using Course and Fine with Montlhy selection
Usually Course and Fine will run every day by default. This code ensure it runs only once a month

```python

def Initialize(self):
    self.SetStartDate(2019, 1, 1)  # Set Start Date
    self.SetEndDate(2019, 12, 31)
    self.SetCash(10000) 

    self.month = -1
    self.AddUniverse(self.CourseSelectionFilter)

def CourseSelectionFilter(self, course):
    if (self.month == self.Time.month): 
        return Universe.Unchanged
    #1. Sort descending by daily dollar volume
    sortedByDollarVolume = sorted(course, key=lambda c: c.DollarVolume, reverse = True)
    #2. Select only Symbols with a price of more than $10 per share
    symbols_by_price = [c.Symbol for c in sortedByDollarVolume if c.Price > 10]
    self.filteredByPrice = symbols_by_price[:8]
    #self.Debug(self.filteredByPrice)
    self.Debug(self.Time)
    self.month = self.Time.month
    
    return self.filteredByPrice
        
```

### Using Course and Fine per Quarter

```python

def Initialize(self):
    self.SetStartDate(2019, 1, 1)  # Set Start Date
    self.SetEndDate(2019, 12, 31)
    self.SetCash(10000) 

    self.AddUniverse(self.CourseSelectionFilter)
    self.update = False
            
def CourseSelectionFilter(self, course):

    if not self.Time.month%3 == 0:
        self.update = True
    
    if not (self.update and self.Time.month%3==0): return []
    

    self.update = False
    #1. Sort descending by daily dollar volume
    sortedByDollarVolume = sorted(course, key=lambda c: c.DollarVolume, reverse = True)
    #2. Select only Symbols with a price of more than $10 per share
    symbols_by_price = [c.Symbol for c in sortedByDollarVolume if c.Price > 10]
    self.filteredByPrice = symbols_by_price[:8]
    self.Debug("quarter! " + str(self.Time))
    return self.filteredByPrice
                
```

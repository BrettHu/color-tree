# ColorTree demo

* Demo prep:
```
git clone https://github.com/codemonkeychris/color-tree.git color-tree
git checkout demo-start
start src\ColorTree.sln
```

Transitions 
It is a common problem where Narrator (screen reader) users get lost in transition.  This is usually caused by assumptions around focus, loading that take long delays (network loads), and not setting focus back to the last list item you were on before you drilled into the list. 
This type of pattern is common across any type of app that goes to the web/external network to get data.  This is true for apps such as music and video apps that have top level selections of groups of generas, then loads movies or music. 
Let us fire up the app and see what happens. 

    
* Baseline accessibility
    * F5
    * Show UX
    * Highlight problems
        * Extra goop read on each line
        * No notification of data loading
        * Focus isn't restore
       
Fire up Narrator via WindowsKey + Enter - (note same key set shuts off Narrator)
Jump into the list, notice it reads funny... hmmm..
Press enter on a list item...
There is a lag but the user is not notified.
From there it is a long dealy and nothing is read.
Go back, same thing, nothing read.
        
* Fix what is read for each item
    * Wire up listener (in constructor)
    ```C#
    gridView1.ContainerContentChanging += GridView1_ContainerContentChanging;
    ```
    * Implement method
    ```C#
    private void GridView1_ContainerContentChanging(ListViewBase sender, ContainerContentChangingEventArgs args)  
    {  
        FrameworkElement source = args.ItemContainer as FrameworkElement;  
        if (args.Item is ColorEntry)  
        {  
            AutomationProperties.SetName(args.ItemContainer, ((ColorEntry)args.Item).name);  
        }  
        else  
        {  
            var i = (KeyValuePair<string, List<ColorEntry>>)args.Item;  
            AutomationProperties.SetName(args.ItemContainer, i.Key);  
        }  
    }  

Note: when it is reading better - that we have the N of M free in XAML. - nice!

Now when we load, let us notify the user that loading is hppaneing.. 

    ```
* Notify user of loading data
    * Add invisible element (not Visibility=Collapsed)
    ```XAML
    <TextBlock Opacity="0" Visibility="Visible" x:Name="readme" AutomationProperties.LiveSetting="Polite" /> 
    ```
    * Add method to update
    ```C#    
    void updateNarrator(string msg)  
    {  
        readme.Text = msg;  
        FrameworkElementAutomationPeer.FromElement(readme).RaiseAutomationEvent(AutomationEvents.LiveRegionChanged);  
    }
    ```  
    * Update user at key points
    ```C#
    updateNarrator("Loading data");
    ```
Now that the user knows the data is loading... do we need to tell them when it is done? 
The simple answer is - no, not if there are items in the list.
We can simply set focus to the list item. Why does this work?
Simpley put we have a start state of: The list item 
A transition of the: "Loading"
and an End State of: Setting focus on teh first item.
Le't smake foucus happne!

* Restore focus (solves a couple problems)
    * Basic notification
    ```C#
    if (gridView1.Items.Count > selectedIndex)  
    {  
        gridView1.ScrollIntoView(gridView1.Items[selectedIndex]);  
        gridView1.UpdateLayout();  
        var item = (Control)gridView1.ContainerFromIndex(selectedIndex);  
        item.Focus(FocusState.Programmatic);  
    }  


Pick a color with nothing in it..

    ```
    * Don't forget special case of zero items
    ```C#
    if (topLevelClicked.Value.Count == 0)  
    {  
        updateNarrator("No data found");  
    }  
    ```
Run it end-to-end now..

What did we do:
1) You know loading 
2) You know when loading is done
3) You know when there is nothing htere
4) You know where you are when we come back.

Run through it with Narrator.
Note - by putting focus back, we end up helping folks start where they left off...
This not only helps Narrator users but it helps Magnifer users.

Fire up Narrator, Full screen with focus tracking on.
Walk through the same demo (Narrator can be on)

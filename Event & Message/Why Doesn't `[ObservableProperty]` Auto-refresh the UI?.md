```c#
[ObservableProperty]
    private ObservableHashSet<string> availableScenarios = databaseService.Scenarios;
```

### Issue Analysis: Why doesn't `[ObservableProperty]` auto-refresh the UI when collection changed?  

The core issue is that the `[ObservableProperty]` attribute in MVVM Toolkit **only triggers UI updates when the property itself is reassigned** (i.e., when the reference to the collection changes). It does NOT automatically react to **changes in the collection's contents** (e.g., adding/removing items), because the property still points to the same collection instance.  


### Simplest Solutions to Refresh the UI  

#### **Option 1: Reassign the same collection (MVVM Toolkit-friendly)**  
```csharp
[ObservableProperty]
private ObservableHashSet<string> availableScenarios;

public MainWindowViewModel(...)
{
    // Initialize and subscribe to collection changes
    AvailableScenarios = databaseService.Scenarios;
    databaseService.Scenarios.CollectionChanged += (s, e) => 
        AvailableScenarios = databaseService.Scenarios; // Reassign to trigger auto-notification
}
```  

**How it works**:  
By reassigning the same collection instance to the property, you force the `[ObservableProperty]`-generated code to fire its auto-generated `PropertyChanged` notification, refreshing the UI.  


#### **Option 2: Use a wrapper collection with built-in notifications**  
```csharp
// Custom collection that triggers notifications on content changes
public class NotifyingObservableHashSet<T> : ObservableHashSet<T>
{
    public event PropertyChangedEventHandler? PropertyChanged;

    protected override void OnCollectionChanged(NotifyCollectionChangedEventArgs e)
    {
        base.OnCollectionChanged(e);
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Count)));
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs("Item[]")); // Triggers UI for item changes
    }
}

// In your ViewModel
[ObservableProperty]
private NotifyingObservableHashSet<string> availableScenarios;

public MainWindowViewModel(...)
{
    AvailableScenarios = new NotifyingObservableHashSet<string>(databaseService.Scenarios);
}
```  

**How it works**:  
The custom collection explicitly raises `PropertyChanged` events for `Count` and item access when its contents change, which WPF/Silverlight bindings automatically listen to.  


#### **Option 3: Use `CollectionViewSource` (WPF-specific)**  
```csharp
private CollectionViewSource _scenariosViewSource;
public ICollectionView AvailableScenariosView => _scenariosViewSource.View;

public MainWindowViewModel(...)
{
    _scenariosViewSource = new CollectionViewSource { Source = databaseService.Scenarios };
    databaseService.Scenarios.CollectionChanged += (s, e) => 
        _scenariosViewSource.View.Refresh(); // Force UI refresh
}
```  

**XAML Binding**:  
```xml
<ListBox ItemsSource="{Binding AvailableScenariosView}" />
```  

**How it works**:  
`CollectionViewSource` wraps the collection and provides a view that can be manually refreshed, triggering UI updates without property reassignment.  


### Which to Choose?  
- **Option 1** is simplest and aligns with MVVM Toolkit conventions.  
- **Option 2** is ideal for reusable collections needing automatic notifications.  
- **Option 3** works best in WPF when you need advanced view features (sorting/filtering).  

All avoid manual `OnPropertyChanged` calls while ensuring the UI updates when `SelectedScenarios` contents change.
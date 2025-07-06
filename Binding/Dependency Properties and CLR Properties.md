### Introduction to WPF Dependency Properties  

**Definition**:  
A dependency property is a special property mechanism in the WPF framework that allows a property's value to be determined by "depending" on other sources (such as bindings, styles, parent element properties, default values, etc.), enabling dynamic calculation and management of property values.  


#### Core Characteristics  
1. **Value Source Priority**:  
   Dependency property values can be provided by multiple sources, with priority from low to high:  
   - Default value → Inherited value → Style/template settings → Data binding → Local value (direct assignment).  

2. **Property Change Notification**:  
   When a dependency property value changes, it automatically triggers UI updates and related logic (e.g., layout recalculation) without manual notification.  

3. **Attached Properties (Special Dependency Properties)**:  
   Allow one object to set properties for another object. For example, `Grid.Row` can be set on any control without the control defining the property itself.  


#### Examples of System Dependency Properties  
- **`FrameworkElement.Width/Height`**: Control width and height, dynamically determined via binding, style, or direct assignment.  
- **`Control.Background`**: Control background supporting colors, gradients, images, etc., set dynamically via resources or binding.  
- **`TextBlock.Text`**: Text content, commonly used in data binding (e.g., `{Binding UserName}`).  
- **`Button.Click`**: Event dependency property for registering click event handlers.  


#### Why Use Dependency Properties?  
- **Resource Sharing and Styling**: Dependency properties can be uniformly set via styles (Style), avoiding code duplication.  
- **Data Binding Support**: Native support for two-way binding, eliminating manual value synchronization.  
- **Property Inheritance**: Some dependency properties (e.g., `FrameworkElement.FontSize`) are automatically inherited by child elements, simplifying layout design.  
- **Dynamic Layout and Responsive Design**: Value changes trigger layout updates automatically, adapting to different devices and scenarios.  


#### Basic Implementation Example  
Custom dependency property `MyProperty`:  
```csharp
public class MyControl : FrameworkElement
{
    // Register the dependency property (must be public static readonly)
    public static readonly DependencyProperty MyPropertyProperty =
        DependencyProperty.Register(
            "MyProperty",           // Property name
            typeof(string),         // Property type
            typeof(MyControl),      // Owner type
            new PropertyMetadata("Default Value"));  // Default value and metadata

    // Wrapper property (for external access)
    public string MyProperty
    {
        get { return (string)GetValue(MyPropertyProperty); }
        set { SetValue(MyPropertyProperty, value); }
    }
}
```  

**Usage**:  
```xml
<local:MyControl MyProperty="Value set via dependency property" />
```  

Dependency properties are a core mechanism in WPF, enabling UI elements to interact flexibly with data and styles. They form the foundation for building complex WPF applications.


### Core Differences Between Dependency Properties and Regular Properties  


#### 1. Storage and Value Sources  
| **Dimension**        | **Regular Property (CLR Property)**                          | **Dependency Property (DependencyProperty)**                  |  
|----------------------|-------------------------------------------------------------|--------------------------------------------------------------|  
| **Value Storage**    | Stored in the object's own field (e.g., `private string _value`). | Not stored directly in the object; managed via `DependencyObject`'s internal dictionary (`PropertyBag`). |  
| **Value Sources**    | Only set directly by the object or externally; value is fixed.  | Values can come from multiple sources (binding, style, inheritance, default values, etc.), with dynamically calculated priority supporting dependency on other data sources. |  


#### 2. Functional Features  
| **Dimension**        | **Regular Property (CLR Property)**                          | **Dependency Property (DependencyProperty)**                  |  
|----------------------|-------------------------------------------------------------|--------------------------------------------------------------|  
| **Data Binding Support** | Requires manual implementation of `INotifyPropertyChanged` for UI updates. | Native support for two-way binding; value changes trigger UI updates automatically (e.g., text box content changes). |  
| **Property Inheritance** | Does not support inheritance; child elements cannot inherit values. | Some dependency properties (e.g., `FontSize`) are automatically inherited by child elements, simplifying layout logic. |  
| **Styling and Templating** | Cannot be set via styles (Style); requires per-control assignment. | Can be batch-set via styles/templates, supporting resource sharing (e.g., unifying button background colors). |  
| **Attached Property Capability** | Not supported; properties must belong to the object's type. | Supports attached properties (e.g., `Grid.Row`), allowing one object to set properties for another. |  


#### 3. Implementation Approaches  
| **Dimension**        | **Regular Property (CLR Property)**                          | **Dependency Property (DependencyProperty)**                  |  
|----------------------|-------------------------------------------------------------|--------------------------------------------------------------|  
| **Declaration**      | Direct field and getter/setter: <br>`public string Name { get; set; }` | Requires registration via `DependencyProperty.Register` for static properties with wrappers: <br>`public static readonly DependencyProperty NameProperty;`<br>`public string Name { get/set; }` |  
| **Metadata Support** | No built-in metadata system; manual attributes (e.g., `[Description]`). | Built-in metadata (`PropertyMetadata`) defining default values, change callbacks, validation logic, etc. |  


#### 4. Application Scenarios  
- **Regular Properties**:  
  Suitable for simple data storage (e.g., `UserAge` in business objects) and scenarios without UI interaction or dynamic updates.  

- **Dependency Properties**:  
  Suitable for WPF control development (e.g., background color of custom buttons), data binding needs (e.g., text box text), property inheritance (e.g., window font), or styling (e.g., uniform control rounded corners).  


#### 5. Example Comparison  
**Regular property implementation**:  
```csharp
public class Person
{
    public string Name { get; set; } = "Default Name";
}
```  

**Dependency property implementation**:  
```csharp
public class MyWPFControl : FrameworkElement
{
    // Register dependency property
    public static readonly DependencyProperty DisplayNameProperty =
        DependencyProperty.Register(
            "DisplayName", 
            typeof(string), 
            typeof(MyWPFControl),
            new PropertyMetadata("Default Display Name", OnDisplayNameChanged));
    
    // Wrapper property
    public string DisplayName
    {
        get { return (string)GetValue(DisplayNameProperty); }
        set { SetValue(DisplayNameProperty, value); }
    }
    
    // Value change callback (unique to dependency properties)
    private static void OnDisplayNameChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        // Automatically triggered on value change, used for UI updates or logic processing
    }
}
```  


#### Summary  
Dependency properties are "enhanced properties" designed for UI elements in the WPF framework. By decoupling value storage from calculation, they enable dynamic binding, style sharing, and property inheritance—core mechanisms for building responsive UIs. Regular properties, conversely, are better suited for pure data storage without UI interaction logic.

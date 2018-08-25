# TABLEVIEWS

## Delete Items
Included (but commented) when a new TableViewController is created

```swift
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            // Delete the row from the data source
            Utils.removeItemAt(position: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
        } else if editingStyle == .insert {
            // Create a new instance of the appropriate class, insert it into the array, and add a new row to the table view
        }    
    }
```

# VIEWS
## Segues
### 1 UI
Control Drag Button to other views

### 2 Pass Values
#### 2.1 Using Global variables
```swift
import UIKit

var globalVariable = "MyGlobalVariable"

class ListViewController: UITableViewController {
	...
```

#### 2.2 Passing values though the segues
In the UI click on the Segue and add an identifier like `"toSecondViewControllerId"`

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
	if segue.identifier == "toSecondViewControllerId" {
		let secondViewController = segue.destination as! SecondViewController // with "force" casting
		secondViewController.mySharedVariable = "New Data From First ViewController"
	}
}

```

#### 2.2 Passing values though the segues with TableViews
`didSelectRowAt`

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
	activeRow = indexPath.row // `activeRow` is what we will send in `prepare(for segue`
    performSegue(withIdentifier: "toSecondViewControllerId", sender: nil) 
}
```

## MAPS
* Create an outlet from the map to the view controller.
* Import `MapKit`
* Extend `MKMapViewDelegate`

```swift
import MapKit

class MapViewController: UIViewController, MKMapViewDelegate {

    @IBOutlet weak var map: MKMapView!
    var dreamLocation  :DreamLocation?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let latitude : CLLocationDegrees = Double(dreamLocation!.lat)!
        let longitude : CLLocationDegrees =  Double(dreamLocation!.long)!
        let location = CLLocationCoordinate2D(latitude: latitude, longitude: longitude)        

        let latDelta : CLLocationDegrees = 0.1 // Zoom Level
        let lonDelta : CLLocationDegrees = 0.1
        let span = MKCoordinateSpan(latitudeDelta: latDelta, longitudeDelta: lonDelta)

        let region = MKCoordinateRegion(center: location, span: span)

        map.setRegion(region, animated: true)
    }
```

### Add Map Annotations
```swift
  let annotation = MKPointAnnotation()
        annotation.title = dreamLocation?.name
        annotation.subtitle = "A dream wave place"
        annotation.coordinate = location
        map.addAnnotation(annotation)
    }
```

### Add LongPress Recognizer  
```swift
    let longPressRecognizer = UILongPressGestureRecognizer(target: self, action: #selector(MapViewController.longpress(gestureRecognizer:)))
    longPressRecognizer.minimumPressDuration = 2
    map.addGestureRecognizer(longPressRecognizer)

    @objc func longpress(gestureRecognizer: UIGestureRecognizer) {
        let touchPoint = gestureRecognizer.location(in: self.map)
        let coordinate = map.convert(touchPoint, toCoordinateFrom: self.map)
        let annotation = MKPointAnnotation()
        annotation.coordinate = coordinate
        annotation.title = "New place"
        annotation.subtitle = "Maybe I'll go here too..."
        map.addAnnotation(annotation)
    }
```

### Add Location Aware    
Add CoreLocation Framwork  

* Go to Main/Build Phases
* Link Binary With Libraries
* Add CoreLocation.framwork  

Add Permissions

* Go to info.plist
* Add new Row
* Add Privacy - Location Always Usage Description
* Add Privacy - Location When In Use Usage Description

Import `CoreLocation`

Extend `CLLocationManagerDelegate` 

```swift
var locationManager = CLLocationManager()
    
override func viewDidLoad() {
    super.viewDidLoad()
    locationManager.delegate = self
    locationManager.desiredAccuracy = kCLLocationAccuracyBest
    locationManager.requestWhenInUseAuthorization()
    locationManager.startUpdatingLocation()
}

func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    print(locations)
    ...
}
```

# CoreData

Import `CoreData`

## Add Data
Get the App Delegate

```swift 
let appDelegate = UIApplication.shared.delegate as! AppDelegate 
```
Get the context

```swift
let context = appDelegate.persistentContainer.viewContext
```
Create the entity object

```swift
let newUser = NSEntityDescription.insertNewObject(forEntityName: "Users", into: context)
newUser.setValue("kirsten", forKey: "username")
newUser.setValue("myPass", forKey: "password")
newUser.setValue(35, forKey: "age")
```
Try catch the save

```swift 
do {
    try context.save()
    print("Saved")
} catch {
    print("There was an error")
}
```

## Get Data

Create a request

```swift
let request = NSFetchRequest<NSFetchRequestResult>(entityName: "Users")

// This is needed to get all the data from the Database
// https://stackoverflow.com/a/23007159/749393
request.returnsObjectsAsFaults = false 
```

Try and catch the request

```swift
do {
    let results = try context.fetch(request)
    if results.count > 0 {
        for result in results as! [NSManagedObject] {
            if let username = result.value(forKey: "username") as? String {
                print(username)
            }
        }
    } else {
        print("No results")
    }
} catch {
    print("Couldn't fetch results")
}
```
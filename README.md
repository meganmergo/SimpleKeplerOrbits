# Simple Kepler Orbits

## First usage guide

	* Create new object, which will be attractor.
	* Create other object, which will be orbiting body.
	* Create third object, which will be velocity handle helper object.
	* Attach KeplerOrbitMover component to orbiting body object.
	* Assign attractor object to AttractorObject field inside KeplerOrbitMover component.
	* Assign velocity helper object to VelocityHandler field inside KeplerOrbitMover.
	* Attach KeplerOrbitLineDisplay component to orbiting body object to be able to see orbit curve.
	* Move orbiting body and velocity helper in scene window and notice how orbit curve changes.
	* If you hit play, body will start moving along orbit curve around attractor body.

## Repository

https://github.com/Karth42/SimpleKeplerOrbits

## Components

	* KeplerOrbitMover - component for making object affected by simulation. It contains main settings and orbit state.
	* KeplerOrbitLineDisplay - component for displaying orbit path via customizable LineRenderer object or via editor Gizmos.

## Scripting

This plugin is designed to be customizable via editor inspector and via scripting. 
Scripting part is not mandatory for functionality, but it provides some additional flexibility for customization.

Main MonoBehaviour component, KeplerOrbitMover, is just an lightweight adapter between unity scene and KeplerOrbitData container. 
KeplerOrbitData container struct contains full orbit state and all orbit handling logic and can be used in scripts even without KeplerOrbitMover in different situations.
This document, however, will give examples only for KeplerOrbitData in pair with KeplerOrbitMover situations.

For detailed algorithms description see [Concept](Docs/Concept.odt) document.

List of some scripting snippets, that can be usefull:

#### Orbit initialization

```cs
	// Initializing orbit via KeplerOrbitMover public interface.
	
	var body = GetComponent<KeplerOrbitMover>();
  
	// Setup attractor for orbit.
	body.AttractorSettings.AttractorObject = attractorReference;
	body.AttractorSettings.AttractorMass = attractorMass;
  
	// Lock inspector editing.
	body.LockOrbitEditing = false;

	// Create orbit from custom position and velocity
	body.CreateNewOrbitFromPositionAndVelocity(newPosition, newVelocity);
```

```cs
	// Initializing orbit from orbit elements (JPL database supported)
	
	var body = GetComponent<KeplerOrbitMover>();
	
	// Setup attractor settings for update process;
	// Note: AttractorSettings is not required for OrbitData setup, but it will be used in Update later, so it is initialized with same parameters.
	body.AttractorSettings.AttractorObject = attractorTransform;
	body.AttractorSettings.AttractorMass = attractorMass;
	body.AttractorSettings.GravityConstant = GConstant;
	
	// Setup orbit state.
	body.OrbitData = new KeplerOrbitData(
		eccentricity: eValue,
		semiMajorAxis: aValue,
		meanAnomalyDeg: mValue,
		inclinationDeg: inValue,
		argOfPerifocus: wValue,
		ascendingNodeDeg: omValue,
		attractorMass: attractorMass,
		gConst: GConstant);
	body.ForceUpdateViewFromInternalState();
```

```cs
	// Initializing orbit from orbit vectors.
	
	var body = GetComponent<KeplerOrbitMover>();
	
	// Setup attractor settings for update process;
	body.AttractorSettings.AttractorObject = attractorTransform;
	body.AttractorSettings.AttractorMass = attractorMass;
	body.AttractorSettings.GravityConstant = GConstant;
	
	// Setup orbit state.
	body.OrbitData = new KeplerOrbitData(
		position: bodyPosition, 
		velocity: bodyVelocity, 
		attractorMass: attractorMass, 
		gConst: GConstant);
	body.ForceUpdateViewFromInternalState();	
```

#### Orbit changing

```cs
	// Make orbit circular.
	body.SetAutoCircleOrbit();
```
```cs
	// Update attrractor settings.
	body.AttractorSettings.AttractorMass = 100;
	
	// Refresh orbit state to apply changes.
	body.ForceUpdateOrbitData();
```
```cs
	// Set different eccentricity for orbit, leaving perifocus and mean anomaly unchanged.
	body.OrbitData.SetEccentricity(newEccentricity);
	
	// Update transform from orbit state.
	body.ForceUpdateViewFromInternalState();
```
```cs
	// Set anomaly for orbit.
	body.OrbitData.SetMeanAnomaly(anomalyValueInRadians);
	// Or other anomalies:
	// body.OrbitData.SetTrueAnomaly(anomalyValueInRadians);
	// body.OrbitData.SetEccentricAnomaly(anomalyValueInRadians);
	
	// Note: changing one anomaly will automatically update other two.
	
	// After changing orbit state, transform should be updated:
	body.ForceUpdateViewFromInternalState();
```
```cs
	// Progress mean anomaly by mean motion, scaled by delta time.
	body.OrbitData.UpdateOrbitAnomaliesByTime(deltaTime);
	body.ForceUpdateViewFromInternalState();
	
	//Note: the mean motion is dependent on orbit period. It will behave differently for different orbits.
	//To strictly set some certain orbit time or progress ratio (independent from orbit state), set anomaly value explicitly instead.
```
```cs
	// Rotate whole orbit by some quaternion.
	body.OrbitData.RotateOrbit(rotation);
	body.ForceUpdateViewFromInternalState();
```
```cs
	//Get orbit points for orbit line in array.
	var array = body.OrbitData.GetOrbitPoints();
	//Non-allocating version of same method, which is more efficient in Update methods.
	body.OrbitData.GetOrbitPointsNoAlloc(ref array, pointsCount: 100, origin: body.AttractorSettings.AttractorObject.transform);
```
```cs
	//Get orbit point at certain anomaly angle, without changing current orbit state.
	//Useful for manual sampling of orbit points.
	var position = body.OrbitData.GetFocalPositionAtEccentricAnomaly(anomalyValue);
```


## Contacts

If you have any questions about this plugin, feel free to write your feedback on **itanksp@gmail.com**

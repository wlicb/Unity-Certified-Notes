

### Chapter 1

#### Week 1: Evaluating Project Requirements

![](C:\Users\Lenovo\Videos\Captures\Pac-Man CCD.pdf - 个人 - Microsoft​ Edge 2020_11_17 10_12_00.png)

![](C:\Users\Lenovo\Videos\Captures\Solution 1 _ Scripting Needs _ Coursera - Google Chrome 2020_11_17 10_15_45.png)



#### Week 2: Basic User Input and Object Management

1. Version Control for Professional Development Teams:

   a. Version control allows the developer to collaboratively edit the same project at the same time.

   b. Several common concepts: 

   - Respository
   - Add
   - Publish
   - Pull
   - Conflict Resolution: Merge
   - History

   c. Can be used in the Cloud menu of Unity and click on Collaborate.

   d. Unity Cloud Build provides a great solution for building the project.

2. Object Management:

   a. To run the game both on mobile devices and the computer: use `CrossPlatformInputManager`for input under the `UnityStandardAssets.CrossPlatformInput` namespace.

   b. To implement physics in 3D game: all the types are the same except that there will not be "2D" at last. (eg. `Rigidbody`)

   c. To avoid the speed changing when both directions have speed: Normalize the `Vector3` vector

   ```c#
   if (vec3.magnitude > 1)
   	vec3.Normalize();
   ```

   d. To change the direction of the game object: we can use `public void LookAt(Transform target, Vector3 worldUp = Vector3.up)`method (the latter parameter is the vector specifying the upward direction.)

   ```c#
   bullet.transform.LookAt(mPos3D);
   
   transform.LookAt(target, Vector3.left);
   ```

   e. To make the game object move, we can change the velocity of the rigid body of the game object:

   ```c#
   rigidbody.velocity = vel * shipSpeed;
   ```

   f. The property `Input.mousePosition` can access the position of the mouse.

   g. `Update` runs once per frame. `FixedUpdate` can run once, zero, or several times per frame, depending on how many physics frames per second are set in the time settings, and how fast/slow the frame rate is.

   It's for this reason that `FixedUpdate` should be used when applying forces, torques, or other physics-related functions 

   h. Using `[RequireComponent(typeof(Rigidbody))]` can make sure that the gameobject that the script is attached to has certain components.

   i. Using Header can enable setting some parameters in the inspector.

   ```c#
   [Header("Set in Inspector")]
   public float shipSpeed = 10f;
   ```

   ![](C:\Users\Lenovo\Videos\Captures\无标题 - Google Chrome 2020_11_19 9_59_52.png)

   #### Week 3: Spawning and Destroying Objects

   1. To create a scriptable object to hold data, a child class of `scriptableObject` can be defined.

      ```c#
      [CreateAssetMenu(menuName = "Scriptable Objects/AsteroidsSO", fileName = "AsteroidsSO.asset")]
      [System.Serializable]
      ```

      The above needs to be in the scriptable object script to make in accessible under the menu.

      Afterwards, when right click the mouse, there will be the scriptable object that is needed to create.

      When creating a game object, the script needs to include 

      ```c#
      public AsteroidScriptableObject asteroid0;
      ```

      in order to make a reference of the scriptable object.

   2. `this` can not be used to assign static objects.

   3. To spawn a game object which is far away enough from the existing game object:

      ```c#
      for (int i = 0; i < 3; i++)
      {
          Asteroid new_asteroid = Asteroid.SpawnAsteroid();
          Vector3 spawnPos;
          do
          {
              spawnPos = ScreenBounds.RANDOM_ON_SCREEN_LOC;
          } while ((spawnPos - PlayerShip.POSITION).magnitude <
          MIN_ASTEROID_DIST_FROM_PLAYER_SHIP);// Assign a random place until the 		   											//distance is large enough
          
          new_asteroid.transform.position = spawnPos;
          new_asteroid.size = asteroidS0.initialSize;
      }
      ```

   4. Use `public void SetParent(Transform p, bool worldPositionStays) ` method to set parent to one object.

      For the second parameter: If true, the parent-relative position, scale and rotation are modified such that the object keeps the same world space position, rotation and scale as before.

   5. `Random.onUnitSphere` returns one random ` Vector3`  position on a sphere with radius 1.

      Similarly, `Random.insideUnitCircle` returns one random `Vector2` position inside a circle with radius 1.

      `Random.insideUnitCircle` returns one random `Vector2` position inside a sphere with radius 1.

      In order to generate a `Vector3` within one unit circle: data type needs to be convert: eg. Obtain a velocity with the random direction of unit vector inside a circle.

      ```c#
      vel = (Vector3)Random.insideUnitCircle - transform.position;
      ```

      Notice that the above are properties of `Random`  instead of methods.

   6. `Random.rotation` returns  one random rotation value of the transform.

   7. `localPosition` is the position relative to the parent object while `position` is the absolute position.

      Similarly, `localScale` is the scale relative to the parent object.

   8. `public static bool Approximately(float a, float b)` returns true when the two float numbers are similar.

   9. `GetComponentsInChildren<type>`will return all the child components with certain type in an array.

      ```c#
      Asteroid[] children = GetComponentsInChildren<Asteroid>();
      ```

   10. Logics to split a game object when it is shot:

       ```c#
       public void OnCollisionEnter(Collision coll)
           {
               if (transform.parent != null)
               {
                   transform.parent.GetComponent<Asteroid>().OnCollisionEnter(coll);
                   // same as the parent object
                   return;
               }
               
               if (coll.gameObject.tag == "Bullet" || coll.gameObject.transform.root.gameObject.tag == "Player") 
                   // Find the right coll object: bullet or player
               {
                   if (coll.gameObject.tag == "Bullet")
                   {
                       Destroy(coll.gameObject);
                   }
       
                   if (size > 1)
                   {
                       Asteroid[] children = GetComponentsInChildren<Asteroid>();
                       for (int i = 0; i < children.Length; i++)
                       {
                           if (children[i] == this || 
                               children[i].transform.parent != this.transform)
       // Make sure that this object is not child of itself and objects with other parents are not included
                               continue;
                           children[i].transform.SetParent(null, true);// Set the child 														seperate from the parent
                       }
                   }
       
                   Destroy(gameObject);
               }
           }
       ```

       `transform.root` returns the topmost transform in the hierarchy.

       Notice that in line 5, the parent game object does not contain the `OnCollisionEnter` Method, so the script component needs to be obtained.



#### Week 4: Tracking and Displaying Application Data

1. Developer testing and debugging in Unity: 

a. Under the Menu of "Window" we can find "Test  Runner", where an "EditMode test" script will be created with the following structure:

```c#
using UnityEngine;
using UnityEditor;
using UnityEngine.TestTools;
using NUnit.Framework;
using System.Collections;

public clas Vector3Extensions_Text {
	[Test]
	public void Vector3Extensions_TestSimplePasses() {
		// Use the Assert class to test conditions.
	}
	
	// A UnityTest behaves like a coroutine in PlayMode 
	// and allows you to yield null to skip a frame in EditMode
	[UnityTest]
	public IEnumerator Vector3Extensions_TestWithEnumeratorPasses() {
		// Use the Assert class to test conditions.
		// yield to skip a frame
		yield return null;
	}
}
```

b. In `Vector3`, there is a `ComponentDivide` method to divide the components over the components of another vector. 

```c#
Vector 3 result = numerator.ComponentDivide(denominator);
```

c. To Show the above result, the methods in the `Assert` class can be called, such as 

```c#
Assert.AreEqual(new Vector3 (4,1,3), result);
```

d. To track the direction that the game object is moving, `Debug.DrawLine` method can be used: prototype: 

`public static void DrawLine([Vector3] start, [Vector3] end, [Color] color = Color.white, float duration = 0.0f, bool depthTest = true)`

example: 

```c#
void FixedUpdate()
{
	if (trackOffScreen) {
		Debug.DrawLine(trackOffscreenOrigin, transform.position, color.yellow, 0.1f);
	}
}
```

`bool depthTest` determines whether the line should be obscured by objects closer to the camera.

`Gizmos.DrawLine` is similar but will also draw line when it is not in play mode.

2. `WaitForSeconds` is used to suspend the coroutine execution for the given amount of seconds using scaled time. `WaitForSeconds` can only be used with a `yield` statement in coroutines.

   In the example: to make the coroutine wait for a few seconds before finding the next possible position:

   ```c#
   // Wait a few seconds before choosing the nextPos
           yield return new WaitForSeconds(PlayerShip.RESPAWN_DELAY * 0.8f);
   ```

   


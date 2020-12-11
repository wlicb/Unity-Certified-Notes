#### Week 1: Particle and Effect System

1. Particle Systems in Unity:

   a. Can be added as a component under "effect".

   b. Interface of the particle system:

![微信图片_20201211184117](C:\Users\Lenovo\Desktop\UROP\学习\Course 2\Application Systems Programming\微信图片_20201211184117.png)

![微信图片_20201211184132](C:\Users\Lenovo\Desktop\UROP\学习\Course 2\Application Systems Programming\微信图片_20201211184132.png)

​	c. Noise in the particle system means that the particle is performing a jumping effect.

​	d. To restrict the emission of the particle within the screen bounds:

​		disable the emission of the particle system when it is out of bounds.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent( typeof(ParticleSystem) )]
public class OnlyEmitParticlesInBounds : MonoBehaviour {
    
    private ParticleSystem.EmissionModule emitter;

	// Use this for initialization
	void Start () {
        // Get the EmissionModule of the attached ParticleSystem
        emitter = GetComponent<ParticleSystem>().emission;
	}
	
	// LateUpdate is called once per frame, after all Updates have been called.
	void LateUpdate () {
        if (ScreenBounds.OOB( transform.position )) {
            emitter.enabled = false;
        } else {
            emitter.enabled = true;
        }
	}

}
```

​	e. To make the particle system tracks another game object during the game:

​		first set the offset of the particle system relative to the other game object in the Start method, then for each frame in the Update method, set the position of the particle system relative to the other game object.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class PositionRelativeToAnotherGameObject : MonoBehaviour {
    public Transform poi;
    Vector3 offset;

	// Use this for initialization
	void Start () {
        offset = transform.position - poi.position;
	}
	
	// Update is called once per frame
	void Update () {
        transform.position = poi.position + offset;
	}
}

```

​	f. In the emission: there are two modes to emit the particle, which can be set in the "rate over time" and "rate over distance" parameters. The former one will emit every second at the set rate and the latter one will only emit when the parent object is moving. 

![微信截图_20201211193424](C:\Users\Lenovo\Desktop\UROP\学习\Course 2\Application Systems Programming\微信截图_20201211193424.png)

​		g. "Size overtime" can set the relative size of the particle using a curve, random or random values between two curves.



#### Week 2: User Progress and Reward Systems

1. Pausing the game:

   a. Pause the game explicitly:

   pass a boolean variable to determine whether to pause or resume.

   ```c#
   public void PauseGame(bool toPaused)
       {
           PAUSED = toPaused;
           if (PAUSED)
           {
               Time.timeScale = 0;
           }
           else
           {
               Time.timeScale = 1;
           }
       }
   ```

   b. Use delegates to handle other behaviors related to pause:

   (1) Set up for the delegate:

   ```c#
   public delegate void CallbackDelegate(); // Set up a generic delegate type.
   static public event CallbackDelegate GAME_STATE_CHANGE_DELEGATE;
   static public event CallbackDelegate PAUSED_CHANGE_DELEGATE;
   ```

   The first sentence set up a new delegate type that can carry methods with `void` return type and no arguments.

   Then set up 2 `CallbackDelegate`s to hold the game state and whether it is paused.

   (2) During the game:

   ```c#
   private void Awake(){
   	...
       GAME_STATE_CHANGE_DELEGATE += GameStateChanged;
       PAUSED_CHANGE_DELEGATE += PauseChanged;
       ...
   }
   
   void GameStateChanged() {
       this._gameState = AsteraX.GAME_STATE;
   }
   
   void PauseChanged() {
       this._paused = AsteraX.PAUSED;
   }
   
   private void OnDestroy()
   {
       GAME_STATE_CHANGE_DELEGATE -= GameStateChanged;
       PAUSED_CHANGE_DELEGATE -= PauseChanged;
       AsteraX.GAME_STATE = AsteraX.eGameState.none;
   }
   
   static public bool PAUSED
       {
           get
           {
               return _PAUSED;
           }
           private set
           {
               if (value != _PAUSED)
               {
                   _PAUSED = value;
                   if (PAUSED_CHANGE_DELEGATE != null)
                   {
                       PAUSED_CHANGE_DELEGATE();
                   }
               }
   
           }
       }
   
       static public eGameState GAME_STATE
       {
           get
           {
               return _GAME_STATE;
           }
           set
           {
               if (value != _GAME_STATE)
               {
                   _GAME_STATE = value;
                   if (GAME_STATE_CHANGE_DELEGATE != null)
                   {
                       GAME_STATE_CHANGE_DELEGATE();
                   }
               }
           }
       }
   ```

   Define methods to set the status of the game.

   At the beginning, set both of the delegates to carry these methods, and in the static methods below, whenever the paused state or game state is reset, both of the delegates will be called.

   And at the last when the game ends, remove the methods from the delegate in the `OnDestroy` method.

2. Implementing game levels:

   a. To set the information for each game level from the inspector:

   A method is define to parse the list of information set in the inspector and is called at the `Start` method at the beginning of the game.

   ```c#
   void ParseLevelProgression()
   {
       // This takes the information from levelProgression and puts it into LEVEL_LIST;
       LEVEL_LIST = new List<LevelInfo>();
           
   	string[] levelStrings = levelProgression.Split(','); // Split different levels with ','
       for (int i = 0; i < levelStrings.Length; i++)
       {
           string[] levelBits = levelStrings[i].Split(':'); // Split level number and level information with ':'
           string levelName = "Level " + levelBits[0];
           string[] asteroidStrings = levelBits[1].Split('/'); // Split different level information with '/'
           int numInitialAsteroids, numSubAsteroids;
           // We should have good data now
           LevelInfo levelInfo = new LevelInfo(i + 1, levelName, numInitialAsteroids, numSubAsteroids); // Create a new levelInfo object to hold information for one particular level
           LEVEL_LIST.Add(levelInfo); // Add the information to the level list
       }
       ... 
   }
   ```

   The information of levels in the inspectors should be enter like the following:

   `1:3/2,2:4/2,3:3/3,4:4/3,5`

   b. To a start a new game level:

   ```c#
   void StartLevel(int levelNum)
   {
       if (LEVEL_LIST.Count == 0)
       {
           Debug.LogError("AsteraX:StartLevel(" + levelNum + ") - LEVEL_LIST is empty!");
           return;
       } // empty level list
       if (levelNum >= LEVEL_LIST.Count)
       {
           levelNum = 1; // Just loop the levels for now. In a real game, this would be different.
       }
   
       GAME_STATE = eGameState.preLevel;
       GAME_LEVEL = levelNum;
       LevelInfo info = LEVEL_LIST[levelNum - 1];
   
       // Destroy any remaining Asteroids, Bullets, etc. (including particle effects)
       ClearAsteroids();
       ClearBullets();
       foreach (GameObject gameObject in GameObject.FindGameObjectsWithTag("DestroyWithLevelChange"))
       {
           Destroy(gameObject); // Destroy all the game objects that should be deleted when the game level changes
       }
       
       // Set up the new game level
       // Set up the asteroidsSO
       asteroidsSO.numSmallerAsteroidsToSpawn = info.numSubAsteroids;
           
       // Spawn the parent Asteroids, child Asteroids are taken care of by them
       for (int i = 0; i < info.numInitialAsteroids; i++)
       {
           SpawnParentAsteroid(i);
       }
   }
   ```

   `ClearAsteroids` and `ClearBullets` are defined to clear all the remaining game objects in the current scene. In this method, they are used to clear all the game objects from the last game level.

   c. To end a game level:

   ```c#
   void EndLevel()
   {
       if (GAME_STATE != eGameState.none)
       {
           PauseGame(true); // Pause the game when one level ends
           GAME_LEVEL++; // move to the next game level
           GAME_STATE = eGameState.postLevel; // Change the game state to postLevel, which means after completing a level, which is defined in the same script
           LevelAdvancePanel.AdvanceLevel(LevelAdvanceDisplayCallback, LevelAdvanceIdleCallback); 
       }
   }
   ```

   d. Two call back methods are defined to deal with the behaviors after the level advancement and in the previous method, these two methods are passed as delegates to the `AdvanceLevel` method defined in the `LevelAdvancePanel` class.

   ```c#
   void LevelAdvanceDisplayCallback()
   {
       StartLevel(GAME_LEVEL); // Start the next level
   }
   
   void LevelAdvanceIdleCallback()
   {
       GAME_STATE = eGameState.level;// Change the game state to level, which means a level is in progress, which is defined in the same script
   
       PauseGame(false); // unpause the game
   }
   ```

   The `LevelAdvanceDisplayCallback` method will initialize the next game level while the information of the next level is being displayed in the scene. The `LevelAdvanceIdleCallback` method will start the next game level when the state of the panel is "idle", which is after all the information is displayed and has faded out.

​		
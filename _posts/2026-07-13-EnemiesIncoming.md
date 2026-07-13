---
layout: post
title: "Week 2 — Enemies Incoming"
---

In the first week I managed to control my player and shoot with it, but a triangle that can shoot was kinda pointless without squares to shoot.

---
## Creating Enemies
I added the squares to the scene and gave them a collider so they could collide with my bullets. I also added another sprite to create an incredibly juicy death animation.

 <details markdown="1">
<summary>Box_death.cs</summary>

```csharp
using UnityEngine;

public class Box_death : MonoBehaviour
{
    //yes i named the animation sprite as corpse
    public GameObject Corpse;

    void OnTriggerEnter2D(Collider2D HitInfo)
    {
        Debug.Log(HitInfo.name);
        Destroy(gameObject);
        Instantiate(Corpse);
    }

}
```

</details>

The problem with this script is obvious now. At my first attempt I did not calculate where to Instantiate the animation sprite, I also forgot to destroy the sprite so the result came out like this.



<p align="center">
  <img src="/assets/Week2/BadDeathAnimation.gif" alt="Death sprite spawning at the wrong position and never disappearing" width="600">
  <br>
  <em>Sprite spawns at the wrong position and never gets destroyed</em>
</p>

---

For the continuity problem I wrote a script that destroys the animation sprite. My first disappear script was also poorly written. I took the Time.time value at the start and gave a destroy command after that time. So the animations couldn't appear after the first second of the scene


<p align="center">
  <img src="/assets/Week2/BadTimedDeathAnimation.gif" alt="Death animations no longer appearing after the first second" width="600">
  <br>
  <em>No animations after the first second</em>
</p>

---

After these attempts I added a public Transform field to the Box_death script and managed to create animation sprites at the center of the enemies. I also managed to calculate the appearance window of the animation sprite by taking the time each sprite was created in its Start() method.


<details markdown="1">
<summary>Box_death.cs</summary>

```csharp
using UnityEngine;

public class Box_death : MonoBehaviour
{
    public Transform AnimationCenter;
    public GameObject Corpse;

    void OnTriggerEnter2D(Collider2D HitInfo)
    {
        Debug.Log(HitInfo.name);
        Destroy(gameObject);
        Instantiate(Corpse,AnimationCenter.position,AnimationCenter.rotation);
    }

}
```

</details>


<details markdown="1">
<summary>Disappear.cs</summary>

```csharp
using UnityEngine;

public class Disapear : MonoBehaviour
{
    private float AppearingTime = 0.0f;
    void Start()
    {
        AppearingTime = Time.time;
    }

    void Update()
    {
        if (Time.time > AppearingTime + 0.2f)
        {
            Destroy(gameObject);
        }
    }
}

```

</details>

<p align="center">
  <img src="/assets/Week2/FirstProperDeathAnimation.gif" alt="First correctly centered and timed death animation" width="600">
  <br>
  <em>First proper death animation</em>
</p>

---

I deleted the enemies from the scene and saved them as prefabs. I wanted to spawn every enemy at the beginning of the scene in a grid format as in the original game.  I had no idea how to spawn the enemies, so I followed [this tutorial](https://www.youtube.com/watch?v=SELTWo1XZ0c).

<p align="center">
  <a href="https://www.youtube.com/watch?v=SELTWo1XZ0c">
    <img src="https://img.youtube.com/vi/SELTWo1XZ0c/hqdefault.jpg" alt="Grid spawning tutorial" width="480">
  </a>
  <br>
  <em>
Unity Tutorial (2021) - Making an Enemy Spawner</em>
</p>

In the tutorial the tutor creates a GameObject and attaches a spawner script to it. In the script he takes the **Transform** of the object and instantiates the enemies at random distances away from the Transform.position . Not quite what I like to do but the logic was simple to copy. I created a GameObject at the center of the scene and attached an EnemySpawner script. In the script I took the column and row amounts and created a grid by using a foreach function with those values.

<details markdown="1">
<summary>EnemySpawner.cs</summary>

```csharp
using UnityEngine;

public class EnemySpawner : MonoBehaviour
{
    [SerializeField]
    private Transform SpawnerPoint;

    [SerializeField]
    private GameObject Enemy;

    [SerializeField]
    private int columnSize;

    [SerializeField]
    private int rowSize;

    [SerializeField]
    private float  columnMultiplier = 0.5f;

    [SerializeField]
    private float rowMultiplier = 0.5f;

    [SerializeField]
    private Transform FormationCenter;

    void Start()
    {
        for(int row =0 ; row<rowSize ; row++)
        {
            for(int col =0 ; col<columnSize ; col++)
            {
                Vector3 spawn = SpawnerPoint.position;
                spawn.y += row*rowMultiplier;
                spawn.x += col*columnMultiplier;
                Instantiate(Enemy,spawn,SpawnerPoint.rotation,FormationCenter);
            }
        }
    }
}


```

</details>

<p align="center">
  <img src="/assets/Week2/SpawningEnemiesIntoGrid.gif" alt="Enemies spawning into a grid formation at scene start" width="600">
  <br>
  <em>Grid formation</em>
</p>

---

## Movement

### Approach 1

Enemies that spawned in a formation were nice but I had to make them move like in the original game of course. Movement was trickier than I expected. I don't know why but my first instinct was adding a collider to the parent GameObject. I guess I just thought it would make my job easier when it comes to changing the direction of the movement each time the formation reaches the edge of the screen. It worked obviously but there was an obvious problem, the edges of the formation were constant, which means even if the player cleared out the first two columns of the right side the formation would not change the boundary of the formation. This led to gameplay like this.

<p align="center">
  <img src="/assets/Week2/EnemiesMovesInFormationBadFormationCollider.gif" alt="Formation changing direction too early because of a fixed-size collider" width="600">
  <br>
  <em>Formation changes direction way too early</em>
</p>

Also there was one more detail that bugged my mind. Since there was a collider at the middle of the formation, I figured the player's bullets could not pass through the middle of the formation. But as seen in the gif above it could.

When I asked about how this could happen to Claude, it introduced me to weird concept of 
<p align="center">
  <img src="/assets/Week2/tunneling_1.svg" alt="Diagram illustrating the tunneling problem, where a fast bullet skips a thin collider" width="600">
</p>

apparently **tunneling** means when your sprite moves at a pace that can skip a thin collider. For example the collider's length of the formation was 0.01 units and the bullet travels 0.04 units each frame. The bullet also has a 0.02 unit length so the bullet wasn't destroyed by the collider just because it was lucky enough to skip the collider. This wasn't causing any issues for my game but I couldn't rely on luck for my game to work properly.

I had to change my approach anyway because of the boundary issues.

### Approach 2

Final approach that I used to move the formation was measuring the boundaries of the formation each frame and deciding when to change direction based on that measurement. 

For this approach I learned the **parent child** relationship and checked each child of the **FormationCenter** where I spawned the enemies earlier.

Not gonna lie it felt tricky to measure the boundaries of the formation. It led to scenes like this.

<p align="center">
  <img src="/assets/Week2/StuckAtRight.gif" alt="Formation getting stuck against the right edge of the screen" width="600">
  <br>
  <em>I don't even know how I managed to make this</em>
</p>

I didn't record every version of the FormationMovement.cs but the successful one was this.


<details markdown="1">
<summary>FormationMovement.cs</summary>

```csharp
using UnityEngine;

public class FormationMovement : MonoBehaviour
{
    private Rigidbody2D rb;

    [SerializeField]
    private float speed = 10f;
    [SerializeField]
    private float SlideDown = 0.2f;
    [SerializeField]
    private float EnemyDeathMultiplier = 0.03125f;

    private int enemycount=0;
    private Vector2 direction = Vector2.right;
    private Vector2 leadingEdgeX = Vector2.zero;
    

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        enemycount = transform.childCount;
    }

    void Update()
    {
        if (direction == Vector2.right)
        {
            leadingEdgeX.x=float.MinValue;
        }
        else
        {
            leadingEdgeX.x=float.MaxValue;
        }
        foreach (Transform child in transform)
        {

            if (direction == Vector2.right)
            {
                if(child.position.x > leadingEdgeX.x)
                {
                    leadingEdgeX.x=child.position.x;
                }
            }
            else
            {
                if(child.position.x < leadingEdgeX.x)
                {
                    leadingEdgeX.x=child.position.x;
                }
            }
        }

    }

    void FixedUpdate()
    {
        float EnemyDeath = (enemycount - transform.childCount) * EnemyDeathMultiplier;
        float RealSpeed = speed+EnemyDeath;
        Vector2 slide = Vector2.zero;


        if (leadingEdgeX.x>7)
        {
            direction = Vector2.left;
            slide = Vector2.down * SlideDown;
        }
        else if(leadingEdgeX.x < -7)
        {
            direction = Vector2.right;
            slide = Vector2.down * SlideDown;
        }
        Vector2 Movement = direction *RealSpeed * Time.fixedDeltaTime;
        rb.MovePosition(rb.position + Movement + slide);

    }

}

```

</details>

I just calculated the leadingX edge considering the direction I was moving. I changed direction in FixedUpdate method whenever leadingX reached the limit I determined.

I also added an EnemyDeath calculator and sped up the formation as each enemy died.

<p align="center">
  <img src="/assets/Week2/FormationMovementCompleted.gif" alt="Completed formation movement with edge detection and speed-up" width="600">
  <br>
  <em>Completed Formation Movement</em>
</p>

I am pretty happy with my progress so far. I learned **a lot** this week and it was realy fun.

Next week I will make the enemies shoot back at me. I mean it is just cruel if you don't let them hit you back, right?

till next time :wave:
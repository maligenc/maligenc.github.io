---
layout: post
title: "Week 1 — Moving and Shooting"
---
## What I did

I decided to make a small clone of a retro game so that I could learn Unity on the job. I picked Space Invaders. I created a MoSCoW analysis so that I would know what I would do and wouldn't do. It wasn't exactly necessary, but I wanted to build up this kind of habit at the early stages of my **career**. I will add the MoSCoW analysis and weekly plan separately.

Four days have passed since I started the project, but I've already completed the first two weeks' tasks.

Obviously, I am getting help from *Claude*, but rather than having it write the code I need, I use it as a teacher.

First, I created a scene and put in an **AMAZING** space-themed photo to match the theme :wink:
![Amazing Background](/assets/Space_Background.png)

After exporting this amazing background, I added my little triangle as a player and surrounded the scene with white walls.

![First Scene](/assets/FirstScene.png)

I added my first script and my player started to move :tada:
<details>
<summary>PlayerMovement.cs</summary>

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    public float speed = 100.0f; // Speed of the player movement
    
    private Rigidbody2D rb; // Reference to the player's Rigidbody2D component
    // Start is called once before the first execution of Update after the MonoBehaviour is created

    void Start()
    {
        rb = GetComponent<Rigidbody2D>(); // Get the Rigidbody2D component attached to the player
        if (rb == null)
        {
            Debug.LogError("Rigidbody2D component not found on the player object.");
        }
    }

    // Update is called once per frame
    void Update()
    {
        Vector2 moveInput = Vector2.zero; // Initialize movement input to zero

        //Left movement
        if (Keyboard.current.aKey.isPressed || Keyboard.current.leftArrowKey.isPressed) moveInput.x=-1f;
        //Right Movement
        if (Keyboard.current.dKey.isPressed || Keyboard.current.rightArrowKey.isPressed) moveInput.x=1f;

        Vector2 Movement = moveInput * speed * Time.deltaTime; 

        rb.MovePosition(rb.position + Movement);
    }
}
```
</details>

The code was fine and all, but there was an issue. While the character moved, it made sudden jumps along the way. It was moving like it was glitched.

After a little conversation with Sonnet, I learned that using physics in Update() is not a sane thing to do.

I fixed the issue by updating the code like this

<details>
<summary>PlayerMovement.cs-updated</summary>

```csharp

using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    public float speed = 100.0f; // Speed of the player movement
    
    private Rigidbody2D rb; // Reference to the player's Rigidbody2D component
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    private Vector2 moveInput = Vector2.zero; // Initialize movement input to zero
    void Start()
    {
        rb = GetComponent<Rigidbody2D>(); // Get the Rigidbody2D component attached to the player
        if (rb == null)
        {
            Debug.LogError("Rigidbody2D component not found on the player object.");
        }
    }

    // Update is called once per frame
    void Update()
    {
        //Left movement
        if (Keyboard.current.aKey.isPressed || Keyboard.current.leftArrowKey.isPressed) moveInput.x=-1f;
        //Right Movement
        if (Keyboard.current.dKey.isPressed || Keyboard.current.rightArrowKey.isPressed) moveInput.x=1f;
        //Stop movement
        if (!Keyboard.current.dKey.isPressed && !Keyboard.current.rightArrowKey.isPressed && !Keyboard.current.leftArrowKey.isPressed && !Keyboard.current.aKey.isPressed) moveInput.x=0f;
    }

    void FixedUpdate()
    {
        Vector2 Movement = moveInput * speed * Time.fixedDeltaTime; 
        rb.MovePosition(rb.position + Movement);
    }
}


```


</details>

I added colliders to the player and the walls so I made sure that the player wouldn't leave the scene.

I created a bullet prefab and coded a script that made an empty game object (which is a sub-object of the player) create a bullet that moves on the Y-axis.

I made sure that the shooting mechanic had a delay of about 0.5 seconds.

<details>
<summary>shootingScript</summary>

```csharp

using System;
using System.Runtime.CompilerServices;
using JetBrains.Annotations;
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.Scripting.APIUpdating;
using UnityEngine.UI;

public class Weapon : MonoBehaviour
{
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    public Transform FirePoint;
    public GameObject Bullet;

    private float fireTime = -0.5f;
    // Update is called once per frame
    void Update()
    {
        if (Keyboard.current.spaceKey.isPressed && Time.time >fireTime + 0.5f)
        {
            Shoot();
            fireTime = Time.time;
        }
    }

    void Shoot()
    {
        //ShootingLogic
        Instantiate(Bullet,FirePoint.position,FirePoint.rotation);
    }

}


```

</details>


<details>
<summary>BulletScript</summary>

```csharp

using UnityEngine;

public class Movement : MonoBehaviour
{
    private Rigidbody2D BulletBody;
    private float speed = 20f;
    void Start()
    {
        BulletBody = GetComponent<Rigidbody2D>();
        BulletBody.linearVelocityY = speed;
    }

    void OnTriggerEnter2D(Collider2D HitInfo)
    {
        Debug.Log(HitInfo.name);
        Destroy(gameObject);
    }
}



```

</details>

By using these scripts, I managed to create this:

![moveandshoot](/assets/Move&Shoot.gif)

This first document came out sort of straight-up cringe, but I hope my writing and markdown skills will also improve along with my game development skills while I work on this project.

Till next time :wave:

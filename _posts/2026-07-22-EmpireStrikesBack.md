---
layout: post
title: "Week 3 — The Empire Strikes Back"
---

I've been procrastinating on this for a couple of days now, but I am making progress on the game faster than I thought and started to get fuzzy about the details of the development steps. So here it goes.

Last week I mentioned that I would give a weapon to the enemies so my game would not promote NPC cruelty. In the third week I also used the **Canvas** for the first time and created a cute little HUD to display the HP of the protagonist.

---

## Enemies Shooting

Since I already added a weapon to the player, adding a weapon for the enemies was pretty much copying the same thing to enemy prefab. 

The problem was when they were going to shoot. It should be random, but how?

 <details markdown="1">
<summary>First enemy weapon script</summary>

```csharp
using Unity.VisualScripting;
using UnityEngine;

public class EnemyWeapon : MonoBehaviour
{
    [SerializeField]
    private Transform FirePoint;

    [SerializeField]
    private GameObject EnemyBullet;

    private int ShootCalculateWindow=0;
    [SerializeField]
    private int MinShootingWindow=0;
    [SerializeField]
    private int MaxShootingWindow=0;
    private float TryShootTime;

    void Start()
    {
        ShootCalculateWindow = Random.Range(MinShootingWindow,MaxShootingWindow);
        TryShootTime = Time.time+ShootCalculateWindow;
    }
    void Update()
    {
        if (Time.time  >= TryShootTime)
        {
            int shoot = Random.Range(0,10);
            if(shoot <= 3)
            {
                Shoot();
            }
            TryShootTime += ShootCalculateWindow;
        }
    }
    void Shoot()
    {
        Instantiate(EnemyBullet,FirePoint.position,FirePoint.rotation);
    }
}

```

</details>

This script was my first attempt. Sure, it made the enemies shoot in a random order, but the problem was that every enemy decided when to shoot using integer random values. Which means every second some of the enemies shot at the same time. It created a weird / unnatural looking shooting order.

<p align="center">
  <img src="/assets/Week3/EnemiesShootAtTheSameTime.gif" alt="Gameplay clip: several enemies in the formation fire at the same moment, in a visible rhythm" width="600">
  <br>
  <em>Enemies shoot at the same time</em>
</p>

---

First, I converted the random numbers into floats rather than integers. This way I made sure the enemies almost never shot at the same time. When it came down to how randomly they would shoot, my GenAI mentor gave me a sweet formula.

$$R_{\text{swarm}} \approx N \cdot \frac{p}{T}$$
> R<sub>swarm</sub> — fire rate · *N* — aliens alive · *p* — chance to fire per roll · *T* — seconds between rolls

I was thinking one bullet from the swarm should be okay, which made the formula look like this

$$  \frac{T\cdot R_{\text{swarm}}}{N} \approx {p}$$

> R<sub>swarm</sub> — fire rate · *N* — aliens alive · *p* — chance to fire per roll · *T* — seconds between rolls

I hard-coded the fire rate as 1.0 in the script and calculated the probability according to this formula.


 <details markdown="1">
<summary>Better random calculator</summary>

```csharp
using Unity.VisualScripting;
using UnityEngine;

public class EnemyWeapon : MonoBehaviour
{
    [SerializeField]
    private Transform FirePoint;

    [SerializeField]
    private GameObject EnemyBullet;

    private float ShootCalculateWindow=0;
    [SerializeField]
    private float MinShootingWindow=0.0f;
    [SerializeField]
    private float MaxShootingWindow=2.0f;
    private float TryShootTime=1;

    void Start()
    {
        ShootCalculateWindow = Random.Range(MinShootingWindow,MaxShootingWindow);
        TryShootTime = Time.time+ShootCalculateWindow;
    }
    void Update()
    {
        if (Time.time  >= TryShootTime)
        {
            float shoot = Random.Range(0f,1f);
            if(shoot <= 1f*(1f/transform.parent.childCount))
            {
                Shoot();
            }
            ShootCalculateWindow = Random.Range(MinShootingWindow,MaxShootingWindow);
            TryShootTime += ShootCalculateWindow;
            
        }
    }
    void Shoot()
    {
        Instantiate(EnemyBullet,FirePoint.position,FirePoint.rotation);
    }
}

```

</details>

Problem was when there was only one enemy left the probability calculation was turning into

$$  \frac{T\cdot R_{\text{swarm}}}{1} \approx {p}$$

I hard-coded T in the code as 1 at first, so the formula basically meant shooting with 100% probability every time the decision window came. I managed to solve this by randomizing the decision window time between 0.0 and 3.0 (the numbers in the scripts are just defaults, I tune the real ones in the Inspector).

With this logic, whenever the last enemy stands in the swarm, it shoots for certain every time the decision window comes, but each decision window comes around every 1.5 seconds (God, I feel like the smartest man alive over such small things :))

I know I didn't do the math perfectly, and my explanation and my grasp of the math are not exactly perfect, but it approximates well enough anyway :sweat_smile:. It gave me a nice starting point to tweak the metrics.

I also added a PlayerLife.cs to keep track of the player's hit points.

 <details markdown="1">
<summary>PlayerLife.cs</summary>

```csharp
using Unity.VisualScripting;
using UnityEngine;

public class PlayerLife : MonoBehaviour
{
    [SerializeField]
    private int HP=0;
    [SerializeField]
    private GameObject HitAnimation;
    [SerializeField]
    private GameObject DeathAnimation;
    private Rigidbody2D rb;


    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    // Update is called once per frame
    void Update()
    {
        if (HP==0)
        {
            Destroy(gameObject);
            Instantiate(DeathAnimation,transform.position,transform.rotation);
        }

    }
    void OnTriggerEnter2D()
    {
        HP -=1;
        if (HP != 0)
        {
            Instantiate(HitAnimation,transform.position,transform.rotation);
        }
        
    }
}


```

</details>

<p align="center">
  <img src="/assets/Week3/EnemiesShootProperlyandPlayerCanDie.gif" alt="Gameplay clip: enemy bullets come down at irregular intervals and the player ship is destroyed" width="600">
  <br>
  <em>Enemies shoot in a proper random manner and the player can die</em>
</p>

This was also the part where I learned a few things about the collision matrix. I created layers named *"BulletAndEnemy"* and *"EnemyBulletAndPlayer"*. I enabled the collision between these two layers so the enemy bullets don't friendly fire.

<p align="center">
  <img src="/assets/Week3/CollisionMatrix.png" alt="Unity Physics 2D layer collision matrix, with the BulletAndEnemy column and the EnemyBulletAndPlayer row highlighted in red" width="600">
  <br>
  <em>Collision Matrix</em>
</p>

---

## HUD

I made the player a killable object in the scene, so I had to show when the player is going to die. I learned what the Canvas is and how I can use it from [this tutorial](https://www.youtube.com/watch?v=uqGkNTFzYXM).

<p align="center">
  <a href="https://www.youtube.com/watch?v=uqGkNTFzYXM">
    <img src="https://img.youtube.com/vi/uqGkNTFzYXM/hqdefault.jpg" alt="Video thumbnail: Player Health System #2, Heart Display UI Unity tutorial" width="480">
  </a>
  <br>
  <em>
Player Health System #2: Heart Display UI (Unity Tutorial)
</em>
</p>

I made a HealtDisplay.cs and attached it to the Player_SpaceShip GameObject. The script showed the amount of HP left by directly reading the "HP" value from the PlayerLife.cs script. This is also the first time I learned how to read values across my own scripts.

 <details markdown="1">
<summary>HealtDisplay.cs</summary>

```csharp
using UnityEngine;
using UnityEngine.UI;


public class HealtDisplay : MonoBehaviour
{
    
    public int healt;
    [SerializeField] private int maxHealt;
    [SerializeField] private Sprite spareShip;
    [SerializeField] private Sprite destroyedShip;
    [SerializeField] private Image[] ships;
    public PlayerLife playerLife;

    void Update()
    {
        if (!playerLife.playerDead)
        {
            healt = playerLife.HP;
            for (int i=0 ; i <ships.Length;i++)
            {
                if(i < healt)
                {
                    ships[i].sprite=spareShip;
                }
                else
                {
                    ships[i].sprite=destroyedShip;
                }
                if (i<maxHealt)
                {
                    ships[i].enabled = true; 
                }
                else
                {
                    ships[i].enabled=false;
                }
            }
        }

    }
}



```

</details>


But attaching the script to Player_SpaceShip led to a quirky little problem. 

<p align="center">
  <img src="/assets/Week3/Last_Healt_Doesnot_out.gif" alt="Gameplay clip: the player is destroyed but the last ship icon on the HUD is still shown as intact" width="600">
  <br>
  <em>Lost ship does not go out</em>
</p>

This error was caused by the code below in PlayerLife.cs. Since I destroyed the gameObject (which is Player_SpaceShip in this context), I also destroyed HealtDisplay.cs with it. So the canvas did not get a chance to update the HUD before the script was destroyed. After moving the script to the canvas itself, I somewhat managed to solve the problem.

```csharp

    void Update()
    {
        if (HP==0)
        {
            Destroy(gameObject);
            Instantiate(DeathAnimation,transform.position,transform.rotation);
        }

    }

```
---

After solving the problem with the health display, I decided to add a score counter to the top right.

 <details markdown="1">
<summary>ScoreController.cs</summary>

```csharp

using UnityEngine;
using TMPro;


public class ScoreController : MonoBehaviour
{
    [SerializeField] private int pointMultiplier=0;
    private TMP_Text Score;
    public Transform formationTransfrom;
    private int initialEnemyCount=0;

    void Start()
    {
        Score = GetComponent<TMP_Text>();
    }

    void Update()
    {
        int killedEnemyCount = initialEnemyCount-formationTransfrom.childCount;
        int score = killedEnemyCount*pointMultiplier;
        Score.text = $"Score:{score}";    
    }

}



```

</details>

I added a TextMeshPro element and added a font from [Google Fonts](https://fonts.google.com/).

And of course I messed up calculating the score on my first attempt. I was taking the enemy count as 0 at the start of the script and subtracting the enemy count in the scene to calculate how many enemies were killed. This led to the point counter starting from negative values.

<p align="center">
  <img src="/assets/Week3/Negatif_Score_Counter.gif" alt="Gameplay clip: the score counter in the top right starts from a negative number" width="600">
  <br>
  <em>Score counter starts with negative value</em>
</p>

The fix was simple and I implemented it fast.

```csharp

    void Start()
    {
        Score = GetComponent<TMP_Text>();
        initialEnemyCount=formationTransfrom.childCount;
    }

    void Update()
    {
        int killedEnemyCount = initialEnemyCount-formationTransfrom.childCount;

```

Taking the enemy count in the Start function made the initial score equal to 0.

---

## First Sprites

I was so sick of seeing the white rectangles and a triangle that I just added some sprites I found on [Kenney.nl](https://kenney.nl/).

<p align="center">
  <img src="/assets/Week3/SpritePack.png" alt="Kenney's CC0 pixel art pack: top-down planes in blue, red, green, yellow and grey, plus UI icons, numbers and terrain tiles" width="600">
  <br>
  <em>Sprite pack I used</em>
</p>

And my little game started to look like this

<p align="center">
  <img src="/assets/Week3/SpritesChanged.gif" alt="Gameplay clip: the game running with the new sprites instead of the white rectangles and triangle" width="600">
  <br>
  <em>New sprites</em>
</p>

I am enjoying this project so much. I am adding small changes and features every single day. But unfortunately I could not keep up with the improvements I made on the documentation side. I forgot to record some of the progress I made and I had to dig into the git commits to remember what I did (and also to find the faulty code), but that was kinda hard. I am aware that this week's document was not as pretty as the previous ones, but it will get better again.

### Next week

Next week I am planning to add an opening screen. I will also create a "Congrats" screen and a "You Lose!" screen and finally close the game loop.

> spoiler alert - I already did.

I will add the stage logic and add some sound effects to enhance the game feel.

I will create my first demo and send it to my friends for the first playtest.


I was planning on releasing the game on itch.io around week 8, but I guess I might be able to release it way sooner. 

Till next time :wave:
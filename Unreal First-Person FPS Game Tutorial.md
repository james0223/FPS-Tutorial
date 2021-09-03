# Unreal First-Person FPS Game Tutorial



## 1. Setup

- Install 4.26 Version
- Install Animation Starter Pack 

- Create project - Games - Third Person

- Open up the blueprint of the character
- Go to Event Viewport window
  - pull the camera all the way to the head
  - Difference with the first person project is that we have a whole body whereas first person pjt only has a floating arms and a weapon
- Compile - Play -> head can be seen



- What we need to do
  - Socket the camera into the head itself
- Go back to viewport and drag the camera into the Mesh in the hierarchy menu
  - This attaches the camera to the body
- However, to make the camera move in line with the head, we need to merge the head with the camera

- In the FollowCamera, there is the Parent Socket in the Details which is set to none initially
  - Search for head
- Now the camera moves along with the head, but is not aligned with the body and is rotated
  - Rotate the camera by 90 degrees
  - drag the camera so that it corresponds with the head

- However, we cannot move left and right with the mouse, so lets fix that
  - Set the Use Pawn Control Rotation to true for Camera
  - select ThirdPersonCharacter and set Use Controller Rotation Yaw to true
- Go to the EpicGames and Add the Animation Starter Pack to the project



## **Casting**

- The action of trying to convert a game object to a certain class
- It can either fail or succeed, depending on the type of the object interacted
- 한 마디로 특정 물체가 무엇인지 아직 모르므로 그것을 특정 클래스인가? 확인해보는 것
  - IF문과 비슷하다고 볼 수 있을듯



## 2. Health / Armor Bar

- In the ThirdCharacter editor go to Event Graph - variables on the left window
- create a new variable by clicking + button
  - Health
  - Armor
- Set their values to floats and press compile
- set their default values as 1



- Now lets get them to show on the screen
- go back to content browser, right click - user interfaces - widget blueprint
- open it up and drag in a progress bar from palette
- create two of them and put them at the bottom-center of the screen

- give them colors to distinguish them
  - go to apperance - Fill color and opacity
  - select colors
  - to see how they work, go to Progress - percent and try changing the value
- binding their percent values to the parameters we have created
  - click bind and create bind
  - pull out from the arrow of Get percent and find CastToThirdPersonCharacter
    - we are casting to third person char because that is where our parameters are actually stored
  - from `As Third Person Character` pull down and type get armor
    - because we have set the value to float already, we don't need to change its type before returning the value
  - Also, pull from the `Object` of `Cast to ThirdPersonCharacter` and find `Get Player Character` so that it can communicate with the player character
  - Repeat this process for each bar



- Press Designer on the top right to go back
- get those bars locked in the screen by going to Anchors and selecting the place you want
- Now to get the bars on the screen
  - the simplest way for now is
  - go to `third person character`  - event graph
  - create an event for begin play by right clicking on screen - find event beginplay
  - drag the arrow and type create widget
  - in the class - select FPSHud, which is the one you have created
  - drag the return value and type add to viewport



## 4. Regenerate Armor / Damage Function

- Regenerating Health
  - In the third person Event graph, create event tick node
  - drag the arrow to create sequence node
    - leave the then1 as blank as we will use it later
  - drag then0 to create `delay`
    - this sets the delay for regeneration
  - we only want to heal armor if its value is below 1, so we use the `branch` node on completed
  - on `condition`, drag and find `float < float` 
    - set the former float to armor param by finding `get armor`
    - set the latter float to 1
  - drag true and find `set armor` 
    - drag armor to find `float + float`
    - drag the armor from the hierarchy and set it to the first one



- Taking damage function
  - go to the window on the left and find Functions - + button - create function
  - set armor
  - drag armor to create `float - float`
    - set the former to armor
    - set the latter to 0.05
  - create a branch to check if the armor has turned into negative values
  - drag true to create set health
    - for the set value, create float + float
      - first one armor, second one health
  - after that, set the armor back to 0



- Creating Pain volume
  - on the content browser, right click - blueprint class - actor
  - name it PainVolume and open it
  - add component of Box collider
  - drag the PainVolume into the screen from the content browser
  - on the editor, expand the boundaries of box collision
  - Now, go to event graph and drag the Event ActorBeginOverlap's Other actor to create CastToThirdPersonCharacter
  - drag its arrow to find Take Damage 5%
  - however, if you check it after compiling, the damage is only taken once
  - to fix this, go back to the Painvolume and create a boolean variable IsInVolume
  - between castto and take damage, create Set volume and set it to true
  - after that, create event actorEndOverlap on the bottom and make it set the volume to false
  - after takedamage 5%, create a branch to see if the player is still in the volume
  - if true, create a delay of 0.5 sec and reactivate takedamage 5%





## 5. Blood Splash Effect

- Create the widget
- animate it so that it fades in and out



- right click - userinterface - widget blueprint - create DamageEffect and open it
- Drag an image into the editor
  - set size x to 1920 and y to 1080
  - set position value of x and y to 0 to get it centered
  - anchor it as the full screen so that it fits the screen regardless of user's resolution
- In the Appearance - Brush, select the image you wish to input
- Go to Color and Opacity and set A to 0
  - Reason is that we don't want it on the screen at all times
- give it a nice distinguishable name



- Create animation, name it fade
- click track and select red vignette one without any other chars

- click + button and select color and opacity
  - set the arrows to 0.5 and give A 1.0 and point arrows to 1.0 and give A 0.0



- Now we need to activate this animation at the beginning of this widget
- Click Graph on the top right
- select event construct
  - this is called when the widget is created
- create `Play Animation` node and get `Fade` to be connected with In Animation
  - Set the num loops to play to 2 or whatever



- Activating the animation when taking damage
- Go to TakeDamage function
  - Drag the final Set and create widget
  - drag the final arrow to create `Add to Viewport`
  - Notice that just creating a widget only creates it inside the memory, not on the screen
  - Therefore, it must be added to viewport to be shown to players
  - For the owning player, create `Get Player Controller`
  - Remember to connect it from the false value of the branch node to get it activated regardless of situation
- 강의대로 하면 memory leak 발생한다 - widget을 생성하고 거기에 애니메이션을 실행시키는 함수를 장착하여 그것을 실행시키는 방식으로 하는 것이 좋은 것 같다
  - 이것을 가지고 수업시간에 소개해줘도 좋을듯



## 6. Setting up Character Animations

- Blend Space
  - Tells the engine what direction and what animations to use when the player is going at what speed and what direction

- Get the assets

- Create a Folder named Character
- get 3d object mesh file into the folder
- create animations folder
- drag them in and set the skeleton to swat_skeleton



- Creating Animation Blueprint & Animation Blend Space

  - In the Character folder, right click - Animation - Animation Blueprint
  - click on the Swat_Skeleton to use it and press ok

  - Name it Swat_AnimBP and open it



- Just putting in animations will only let you replay single animation over and over again
- to use multiple animations based on different situations, create a state machine and connect it to the Output Pose and open it
- From the Entry, drag it and create a new state called Idle
- Again, drag from it and create a new state named Walk_Run
- Since we should be able to change state from either states freely, draw a new arrow from Walk_Run to Idle



- Before allowing the figure to walk and run, we need to be able to adjust the animation depending on speed and direction

- Go back to content browser - right click - Animation - Blend Space - select the skeleton - name it WalkRun_BS - open it
- On the left, go to Axis Settings
  - Name the Horizontal Axis Direction
  - Name the Vertical Axis Speed
- Set the Maximum speed to 600 as it is the max value determined inside the char blueprint
- For horizontal
  - set Min Axis val to -180 and max to 180
- Set interpolation time to 1 as if you set it to 0, the animations does not get animated smoothly



- Drag the walk_forward_inPlace to the very center of the blend space
  - This is the medium speed with the medium direction(front)
- place run_forward_inPlace at the mid-top
- place walk_backward_inPlace at the mid-far-right and mid-far-left
- walk_left_in place to the mid-left and walk_right_inPlace to the mid-right
- Whatever just see the charts



- Hold down Shift to preview the animations

- When mistakes are made, clikck the middle tag like button and names will show up
  - right click on them to change the animations set



- Go back to Swat_AnimBP and double click Walk_Run state
- Drag the newly created WalkRun_BS into the editor and connect it to the result
- Drag the Direction and Speed and promote to variable
- The values for these vars needs to be calculated in the Event Graph



- From Event Blueprint Update Animation, create sequence and Cast to Character
  - Reason is that we won't be using third person char later on
  - Drag the object and create Try Get Pawn Owner
    - When dealing with animations, we need data to accurately display animations. For example, how fast the character is moving (aka velocity), what is the orientation of the character, is he crouching, is he supposed to be dead, etc. The animation blueprint is always attached to a skeletal mesh component. Since it is a component, it will obviously be attached to something (an actor) as well. You try to get this "something" with the Try Get Pawn Owner node.
  - fro trygetpawnowner, drag out getVelocity and getActorRotation to find out the direction and speed of the character
  - create CalculateDirection on the background and connect it to Set Direction
  - then, create SetSpeed and get the speed by connecting VectorLength to Get Velocity
  - When you compile now, you get the eror `Idle to Walk_Run will never be taken, please connect something to Can_Enter_transition`
    - This error means the engine needs to be told when to enter walk from idle, or vice versa



- Go back to SWAT_AnimBP > Anim Graph > Walk_Run and open up the arrow that connects to Walk_Run
  - create float >= float
  - first float is the speed var and second maybe 10
- go to the other arrow
  - create float <= float
  - first val is the speed var and second the one set above
- Go to Anim Preview editor on the bottom right and try things out



- Now we need to get the character we have created into the game
- Go back to Content browser and open up third person character
- Select Mesh on the top left and set its Skeletal Mesh to Swat
- For Anim Class Select Swat_AnimBP_C
- Scale the model in the viewport so that it fits the Capsule
- Reset the Camera position and rotation



## 8. Weapon

- Find the Obj file for the weapon mesh, normal, texture etc
- In the Content browser, Create a Weapons folder inside Blueprints
- In the Weapons folder, create AK47 folder and open it
- Import the AK47 obj file and make sure to import it as skeletal mesh - no need to give it skeleton for now
- drag the textures into the folder as well
- Delete the Mat file created and create a new one named AK47_Mat and open it
- create a texture sample by t + click
  - set the AK_diffuse as its texture and connect its RGB to base color
  - Do the same steps for normal, specular
  - Specular defines how smooth the surface is



- To make this model work as a weapon
- Create a Blueprint named Weapon_Base
- Add Component named Skeletal Mesh
- set the Skeletal Mesh of Skeletal Mesh to AK



- Now we need to create a socket in the character for the weapon
- open Swat
- go to Skeleton
  - Find out which part of the body to socket the weapon to
  - Since Right hand will always be attached to the weapon, it seems like the best place
  - find RightHand and right click - add socket
  - right click the created socket and add asset - AK47
- To get the gun into the right place
- open up preview scene setting next to Details
- select use specific animation in preview controller and set it to idle
- scale, rotate, transform time to get the gun into place



- Now we need to get the weapon spawned in the scene
  - open up the blueprint of ThirdPersonCharacter
  - Create SpawnActorClass on Event Begin Play and select Weapon_Base as the class
  - on the spawn transform, create Make Transform
  - Drag from the Return Value to create AttachActorToComponent
    - Set its parent to the Mesh in the hierarchy and Socket name the name we had given to the gun socket - has to be the exact name



- When you press play, you can see the gun getting clipped sometime when the camera gets near
  - to fix this, go to project settings - general settings - near clip plane - decrease its value maybe to 1?



## 9. Firing guns

- Create an actor blueprint named Projectile_Base

  - Add Component - Static Mesh

    - Set its skeletal mesh to capsule
    - scale it down to 0.3

  - Add Projectile Movement

    - This does the physics for the bullet - flying, gravity, etc

    - Set its initial speed and maximum speed to 3500
    - set the gravity value to 0.05 as bullets need to travel far



- Now to make the gun fire this bullet
  - Find the blueprint of the weapon, which is Weapon_Base
  - create custom event inside event graph, name it fire
  - drag it out to create spawnActorFromClass
    - Set Projectile Base as its class
    - do not make transform for now
  - Open up AK47 Model
    - go to skelton
    - create socket named muzzle
    - set its position to the muzzle
  - Go back to Weapon_Base
    - get the reference to the skeletal mesh
    - drag to create GetSocketTransform
    - input its correct name and get the returned value to spawned transform



- To receive firing inputs from the user
  - Go to project settings - input - Bindings
  - Press + in Action Mappings and name it PrimaryFire
  - Select the button you wish to respond to - in our case mouse left button



- To get the character to fire the gun on mouse left click
  - Go to ThirdPersonCharacter
  - in the Event Graph, right click, find PrimaryFire
  - to get the reference, promote the return value of SpawnActor Weapon Base to variable and name it EquippedWeapon
    - Don't forget to make it happen between Spawning and Attaching



- If you slow down the bullets and watch them being fired, they go to strange direction
- rotate the bullet in the AK47 skeletal model

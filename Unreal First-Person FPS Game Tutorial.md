# Unreal First-Person FPS Game Tutorial

- Pawn vs Character
  - Pawn is an actor that can be possessed and receive input from a controller
  - Character is a type of Pawn that includes the ability to walk around
    - Character Movement Component allows this to happen, which only resides within characters, not pawns
  - 

- Delta time
  - time elapsed from the frame right before



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





## 10. Crouching with animations

- Create a blend space
  - right click on the Content Browser - Animation - Blend space
  - Select Swat_Skeleton and name it Crouch_BS
    - The goal of a Blend Space is **to reduce the need for creating individual, hard-coded nodes for blending animations** with an asset that performs blending based on specific properties or conditions instead.



- Set Idle_crouch_aiming at all the dots on the bottom
- Change the names of horizontal, vertical to Direction, Speed
- -180 and 180 for min max values of horizontal
- 300 for max axis for vertical because crouching is slower than walking, running



- Drag the walk_crouching_forward_inplace into mid top
- walk_crouching_forward_left for the dot on the left and walk_crouching_forward_right for the dot on the right
- walk_crouching_backward_inplace for each dots on the top right



- Press shift to test it



- To use this Animation we have created
- Go back to the content browser and open up the Swat_AnimBP
  - Go to Walk_Run
  - Drag out from Idle add State Crouching
  - also drag from Walk_Run to Crouching and vice versa
  - Open up the Crouching state
  - Drag Crouching_BS into editor and connect it to the result
  - Connect the previously created Direction and speed variables to the Crouch_BS's slots



- Now the engine needs to be told when to enter crouching
  - Open up the ThirdPersonCharacter in the Content browser
  - Create a new boolean variable called CrouchTrue



- Go to Project settings to designate a key for crouching
  - Input - Action mappings - + - name it Crouch



- Go back to ThirdPersonCharacter
  - In the Event graph, right click to find Crouch in Action Events
  - Create Two SetCrouch and connect them to pressed and released
  - set one to true, other to false



- Also, we need to change the speed when changed to crouching, as the speed value is likely going to exceed the 300 value
  - Grab the reference to CharacterMovement and drag from it to create set max walk speed
  - press ctrl w to copy it
  - when set to true, set the max walk speed to 300 and 600 for false



- Now we need to hook it up into the Anim_BP
  - Go back to Swat_AnimBP
  - We need the reference to the CrouchTrue value from the character
  - In the event graph's sequence
    - create cast to third person character
    - object will be try to get pawn owner
    - as third person character - **get CrouchTrue from Default ** not from Character
    - promote it to variable
  - Go Walk_Run and open up Idle to Crouching arrow
  - get Crouch true and create a node of boolean
  - connect crouch true to the former slot and check the box in the latter
  - do the things for other arrows



## 11. Sprint Systems with animations

- Create a new Blendspace for sprinting
- open it and place running animations on the bottom and sprinting on the top



- we need to create a variable to tell the engine if we are sprinting
- go to third person char and create a var SprintTrue
- Go to input settings in the project settings and set the left shift key to Sprint
- In the Event Graph, create Action event of Sprint
- Do the similar things to Crouch except the speed part
  - max to 1000 and reset to 600



- Go to Swat_AnimBP
  - create the sprint node and set it up just like crouch and walk
  - In the Event graph, create a reference to Sprint true from third person char using the crouch sequence node
  - leftover steps are just like setting up crouch
- Make sure to give an interpolation value of 1 to smooth things up, both for crouch and sprint



## 12 Control rotation

- Make the gun follow us when we rotate our body

- Open up the SWAT_AnimBP
  - we will change the spine position
  - Drag the Walk_Run far from the final animation pose to create some space
  - rightclick to find Transform modify Bone
    - This allows us to modify the transform of the bones
    - there are three spines, and we will modify all three to make a smooth spine rotation
  - Because we are making changes to three spines, duplicate the node two times
  - Since we are only modifying the values of rotation, let's disable the translation, scale, alpha
  - Set each node's bone to modify to spine, spine 1, spine 2
  - connect from Walk_Run to first node sequentially to the Output pose
  - drag out from the first node and promote to variable
    - name it AimRotation
    - connect the AimRotation to all the leftover nodes
  - try moving the character after compiling and nothing will have changed by then because there is no value hooked to the AimRotation



- Go to Swat_AnimBP > Event Graph and create CastToThirdPersonCharacter from the sequence node
  - trygetpawnowner is the target object
  - as third person char, create get control rotation
    - need to study what control rotation is
  - now we need to turn this into AimRotation
  - create a Break Rotator
    - This breaks down the ration values 
    - 언리얼은 왼손 좌표계
    - x축 회전은 Roll
    - y축 회전은 Pitch
    - z축 회전은 Yaw
  - leave x and z and create three nodes from y
    - float - float
    - float x float
    - float > float
  - Create a Select Float node
    - connect minus node to A and multiply node to B and greater to Pick A
    - divide the returned value to 3 by float divide
    - create make rotator node and connect its value to x
    - connect it to the Set Aim rotation node and connect the cast to Third to Set Aim rotation node

  

- now, to create sounds when firing a gun 

  - from the starter content select explosion 01
  - open up projectile_Base
  - add component - audio, which should have selected the explosion01 that we have chosen
  - go to volume multiplier and set its value to 0.3





## 13. Making the gun automatic

- Open up Weapon_Base
- we will create a weapon fire rate and ammo capacity
  - Create a variable named Ammo as integer
    - compile it and set its default value to 50
  - Create a variable named FireRate as float
    - means how much it can fire a gun in a second
    - it is basically the delay between each fire
    - compile it and give it a default value of 0.1



- Go open up the third person character - event graph - InputAction PrimaryFire
  - we need to create a loop in here
  - create a new variable set this to isFiring and create a set isFiring node
  - connect InputAction PrimaryFire to Set Isfiring node
  - duplicate the is firing node and connect it to InputAction PrimaryFire node's Released
    - For pressed, set isFiring to true and false for released
  - connect the set isFiring to fire and from the Fire, create Delay node and set its duration to Fire Rate from the equipped weapon
  - Create a branch that follows the delay and set the condition to isFiring and make a loop into Fire node



- To limit the bullets to Ammo
  - Go to Weapon_Base's Fire function
  - from SpawnActor Projectile Base, create Set Ammo node
  - create get ammo node and connect it to the integer - integer node
  - set it up so that the ammo reduces by 1 each time this function activates



- Open up the ThirdPersonCharacter and create a branch node that checks if Ammo value is greater than 0 and in case it is true, connect it to the Set isFiring



- To show how much Ammo you have left
- Go to FPSHUD
  - Drag in a text
    - change its font size to maybe 50
  - click on bind on the text and a get Text0 window will open
  - cast to third person character
    - the object will be Get Player Character
  - as third person char - get equipped weapon and from equipped weapon get Ammo and connect it to the return value





## 14. Aiming Down Sights

- learn how to create a second camera for aiming and slow down the character while aiming



- Open up the ThirdPersonCharacter
- inside Mesh, create Skeletal Mesh
  - set its parent to Weapon_Attach
  - set its mesh to AK47
  - check its visible and Hidden in game to true
  - scale, rotate the mesh to fit the hands - does not have to be accurate as it will not be shown in the scene
- In the Skeletal mesh we have created, create a camera and name it ADSCamera
  - set its parent socket to the muzzle - this will make the camera move with our weapon
  - rotate and transform the camera so that it will 
    - to be more precise with this camera, scale it down
    - turn off auto activate so that it is not activated when starting the game
  - Set its Use Pawn Control Rotation to true to prevent camera from moving up and down while aiming



- setting up aim function
  - go to project settings to create new input - right click
  - Name it AimDownSights
    - set it to Right mouse button

- Go to thirdPerson character
  - In the events graph, create a node by finding Aimdownsights
  - drag in the FollowCamera and create a Deactivate node
  - Do the opposite for the release



- If we go into the game, you can see that the view is a bit distorted even though we kind of set up the camera view
  - This is because the weapon socket in our third person view is not the same one that gets spawned in the game
  - To solve this, try moving and rotating the camera until you get a nice sight



- now, since the person is moving too quickly when aiming, let's slow it down
  - Go to ThirdPerson Character - Event Graph and create Character Movement node
  - 수업 때는 aiming 모션을 넣어서 해보자





## 15. Spawning Muzzle Flash

- Will use sockets to do this



- Download Infinity Blade: Effects
- add it to the project
- go into InfinityBladeEffects and search for Muzzle and confirm you have a Muzzle Flash



- Open up Weapon_Base
- We will be spawning it in a socket
- In the Event graph, create Spawn Emitter at Location node at the end of the fire function
  - The template should be muzzle flash
  - For the rotation and location, we need to get a reference to the character as we can't directly access the weapon right now
  - so before spawn emitter, create cast to thirdperson character
    - object will be get player character
    - as third person char, create get equipped weapon
    - to find the weapon socket, create Get Socket Location(SkeletalMesh)
    - the Socket Name in Get Socket Location should be Muzzle, as that is the name we have given to the socket in the AK



- When you try shooting, the size would be too big and direction in the wrong way

- To fix this create a Set World Scale 3D node
  - This node allows you to change the scale based on the world
  - change this to 0.1 maybe



- now to adjust the direction
  - create Get Socket Rotation and connect it to the rotation value of the Emitter
    - don't forget to give it the correct socket name





## 16. Fix sprinting and crouching

- To prevent sprinting while crouching and vice versa, set the other to false when activating one
  - for example in the input crouch, set Sprint True to false before starting the actions





## 17. Reload function for Rifle

- When pressing r it will reload



- In Weapon_Base
  - Create MaxAmmo and Clipsize int Variable

- Create a Reload function
  - create a branch to see if a player has a full clip
  - set default values of Max Ammo, Clipsize to 100 and 25



- Go to FPSHUD
  - add a text for max ammo
  - increase the font size and put it in a nice location
  - click bind to create a function that can get the max ammo value
    - just copy and paste the functions for tex0 and change the get ammo to get max ammo



- To use the Reload key
  - Go to project settings input
    - create Reload and bind the R key to it
  - Go to thirdperson char
  - create node for reload
  - drag Equipped weapon and call on reload function and connect it to pressed





## 18. Conservative Reload system

- I did it





## 19. Reload Animation

- Get the Reloading animation
  - open it up and we can see that the animation doesn't really go well with the gun
  - we can fade it later on to make it better
- Go to Swat_AnimBP - Anim Graph - open up Walk_Run - Create new state called reloading
  - open up the Reloading state and drag reloading animation into the editor



- Now we need to create some variables to check if we are reloading or not

  - Open up third person character
  - Go to the Reload function
    - we don't have a variable that can set up the reloading state so go to thirdperson character
    - create a bool var named IsReloading
  - go back to reload function in weapon_base
  - create Cast to Third person char in the end and set the object to Get player Character
  - from the cast node, create set IsReloading node and set it to true
  - we need to set is reloading to false after the animation ends, but since this is a function, the delay cannot be created in this place
  - Therefore, we create the delay outside the function
  - create add return node in the end and drag the return value of set IsReloading into it
    - This allows you to make this function return some specific value

  

- Go to Third Person Char - find reload node in Event Graph and you will see the function returning something

  - create a branch to check if it has returned true
  - in case it has returned true, create a delay node and connect it to set IsReloading - False
  - To find out how long it needs to wait, go open up the reloading animation and find out how long the animation runs



- To set up transitions for reloading
  - We need reference to IsReloading, so go to EventGraph and select any node with Cast to ThirdPersonChar and as Third person char - get Isreloading - promote to variable
  - Use this variable to transit reloading animation





## 20 Fixing Firing system

- Right now, we can fire guns even when we are reloading or sprinting



- To fix this, open up third person char and find the inputaction primary fire
- before activating, make it check the isReloading and isSprinting var





## 21 Aiming with a Crosshair

- Prepare a Crosshair png file



- Open up FPSHUD
  - drag in a Image
  - set its anchor to the middle
  - make its size values equal, such as 100, 100
  - try to move it into the anchor located in the center
  - to make it visible only when not aiming, create an Aiming var in Thirdperson char and set its value depending on AimDownSights in the EventGraph
- reference this value in the FPSHUD - visible - bind - cast to third person char ~~





## 22 Dynamic Spread Crosshair

- Creating a crosshair that moves dynamically with the player character
- for uassets, you cannot directly drag it into the project browser
  - need to manually move it into the content - thirdpersonbp - blueprints
  - it will appear with the name of WBCrosshair



- we will be changing the crosshair_spread based on player's movement
- go to FPSHUD and delete the original Crosshair image
- Go to ThirdPersonChar - EventGraph 
- at the end of creating FPSHUD widget, make a create widget node with WBCrosshair
- get the return value to be promoted into a variable with the name of CrosshairRef
  - The created variable allows us to access its Setcrosshair Spread etc
- then drag it to add to viewport
  - The crosshair should be too small at the beginning, so go back into WBCrosshair to change teh default values of length and thickness



- Now to make it change depending on user's speed
- Go to Third person Character - Event Graph - Event Tick
- Drag the CrosshairRef from the variables and drag from it to create set Crosshair Spread
  - First, create Get Velocity to get the velocity of this third person char
  - Then, create a VectorLength node to turn it into a float variable that we can use
  - Afterwards, create Map Range Clamped and connect its return value to the crosshair speed
    - Map Range Clamped allows you to map or reassign a number from one range of numbers to another range
      - In Range A and B map to Out range A and B based on the given value
      - it moves the value's position from wherever it was in the first range to the new position in the range of the second two relative to the first value
        - let's say your hp is from 0 to 100 but you want to move it to the progress bar which ranges from 0 to 1
        - by using Map Range Clamped, if your hp is 50, you get 0.5
      - The difference between Clamped and Unclamped is whether you can surpass the given range
        - 110 will become 1.1 in unclamped, but 1.0 for clamped
    - The values may vary but depending on the original:
      - In Range A 0.0
      - In Range B 450
      - Out Range A 5
      - Out Range B 80





## 23. Pick up AMMO

- Create a Blueprint Class that a player can collect to replenish AMMO

- In the Content Browser, create a new blueprint class - Actor and name it RifleAmmoPickup
  - Add component of StaticMesh and give it a Mesh
    - Since I don't have Any mesh prepared, maybe give it a Cube mesh
    - scale it to an appropriate size
  - Initially, you won't be able to go through this
    - Go to collision - BlockAllDynamic to Overlap all
    - Overlap allows to activate collision event later on
  - Go to Event node of RifleAmmoPickup
    - on Event ActorBeginOverlap - CastoThirdPersonChar
    - asThirdPersonChar - get EquippedWeapon
      - from Equipped Weapon Get Max Ammo
    - create a Set MaxAmmo from asThirdpersonChar
      - connect its target node with Max Ammo via int + int
    - Create Destroy Actor no the end
    - Create Spawn Emitter and Spawn Sound for more details afterwards





## 24 Setting Up AI & Bullet Damage

- Create a new blueprint for the AI
  - Blueprint - Character - name it SimpleAI - and open it
  - select Mesh on the left and give it the SK Mannequin Mesh - the first one
  - select the Animation - Use Animation Blueprint - ThirdPerson_AnimBP_C
  - transform, scale to make him fit inside the collision capsule
  - rotate him so that he faces the arrow
  - Go out and put him inside the scene



- Now to give him health and stats
- Go back to SimpleAI
  - Create a Health variable as float
  - on Event ActorBeginOverlap - Cast to Projectile_Base
  - create print string that help us check if it is working
- if you go back to the scene and play it, the printed word does not show up
- this is because we need to change the collision settings for the Projectile_Base
  - Open it up - StaticMesh - Collsion 
    - Collision Presets to OverlapAll
    - Generate Overelap Events to true
    - this will generate overlap for this
  - If you test it now, the print words should show up
    - generate overlap for SimpleAI too - Maybe not





## 25. Enemy AI Following/Chasing

- Need to set up a code to make the AI follow the character
- Also need to create a navigational mesh areas where AI is not allowed to go in



- In the Ue4 main page, go to Volumes - drag in Nav Mesh Bounds Volume
- Extend the volume so that it covers the floor and not the floor above the stairs
  - Press P to show the areas covered by it



- To add AI into the SimpleAI
  - open SimpleAI and addcomponent - PawnSensing
    - This allows the AI to sense the player
  - Press compile and click on the Pawn sensing
    - the green radius you see around the mesh is the area that can detect a player
  - Use Peripheral Vision angle and Sight Radius to adjust the range to your taste



- On the bottom, press On See Pawn to designate actions that it will execute on seeing a pawn
  - cast to third person char form the pawn hole
    - test if it is seeing us by printing hello
  - create a node called AI move to
    - Pawn is the target we need to make it move
    - Drag from it and select self
    - As for the destination, we can either select an actor itself or select a location via Destination hole
      - In our case, make it follow actor





## 26 Fixing Aiming

- Fix the part where you can aim while reloading
- Fix the part where you can aim while sprinting
  - for these two create a branch to see if it is reloading
  - if not, copy the cancellation process of sprinting and paste it here before moving on to aiming
  - Also need to fix the sprint part, as you can sprint while aiming down
    - Press Alt+Click to delete a link
    - Make the cancellation part of Aimdown into a function and name it DeactivateAiming
    - copy the function and put it in the true part of the branch and connect it to the afterpart
- remove the dynamic crosshair when aiming
  - create Set Visibility of CrosshairRef node at the end of each aiming node ends and set it to either visible or hidden
  - Another way of doing this is to select all 4 parts in WBCrosshair and bind their visibility to a function that changes its visibility depending on the ThirdPersonChar's Aiming Var



## 27 Smarter AI Actions


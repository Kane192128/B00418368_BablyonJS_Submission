# Element 5 Code Documentation

When creating my Element 5 I included; A way for each scene to reset when changing between them, 
Implementation of a player model and animations that are controlled by WASD and E (sprint), 
Creation of meshes with shadows at runtime, 
UI elements displaying a timer which ends the scene once it hits 60 and a counter for how many times the player gets hit.

-------------------------------------------------------------------------------------
## Scene Reset Code
```typescript
//This is used to set scenes back to their original versions
    export function ResetScenes(i: number){
        if(i == 0){
            scenes[i] = MenuScene(eng);
        }
        else {
            scenes[i] = GameScene(eng);
        }
    }
```
This code is used to reset each scene back to their default state, it is called alongside the 'setSceneIndex' function.

## Scene Change Code
```typescript
 function createSceneButton(scene: Scene, name: string, index: string, x: string, y: string, advtex){
    const buttonClick = new Sound("MenuClickSFX", "./audio/menu-click.wav", scene, null, {
      loop: false, 
      autoplay: false, 
     });

     var StartButton = GUI.Button.CreateSimpleButton(name, index);
     StartButton.left = x; 
     StartButton.top = y; 
     StartButton.width = "160px"
     StartButton.height = "60px"; 
     StartButton.color = "white"; 
     StartButton.cornerRadius = 20; 
     StartButton.background = "green"; 
     StartButton.onPointerUpObservable.add(function() {
      buttonClick.play();
      if (index == "Start Game"){
        setSceneIndex(1); 
        ResetScenes(1);
        advtex.removeControl(StartButton); 
      }

     });
     advtex.addControl(StartButton); 

     return StartButton; 
    }
```
This code is used to set up the start button in the Main Menu, and also disables the button from being activated again 
and also shows utilization of the 'ResetScenes()' function used.


## Character Creation and Control Code

```typescript
let walkspeed: number = 0.1;
    let currentspeed: number = 0.1;
    let runspeed: number = 0.3;
  
    function importPlayerMesh(scene: Scene, child: Mesh, x: number, y: number) 
    {
      let tempItem = { flag: false } 
      let item: any = SceneLoader.ImportMesh("", "./models/", "dummy3.babylon", scene,
      function(newMeshes, particleSystems, skeletons) 
      {
        let mesh = newMeshes[0];
        let skeleton = skeletons[0];
        
  
  
        skeleton.animationPropertiesOverride = new AnimationPropertiesOverride();
        skeleton.animationPropertiesOverride.enableBlending = true; 
        skeleton.animationPropertiesOverride.blendingSpeed = 0.05;
        skeleton.animationPropertiesOverride.loopMode = 1; 
        let walkRange: any = skeleton.getAnimationRange("YBot_Walk");
        let runRange: any = skeleton.getAnimationRange("YBot_Run");
        let idleRange: any = skeleton.getAnimationRange("YBot_Idle");
  
  
         let WalkSpeed: number = 0.03;
         let Sprintspeed: number = 0.06;
         let Currentspeed: number = 0.03;
         let speedBackward: number = 0.01;
         let rotationSpeed = 0.05;
  
         
         let idleAnim: any;
         let walkAnim: any;
         let runAnim: any;
         let animating: boolean = false;
         let isPlaying: boolean = false;
         let isRunning: boolean = false;
  
        scene.onBeforeRenderObservable.add(()=> 
        { 
          let keydown: boolean = false; 
          let shiftdown: boolean = false; 
  
          if (keyDownMap["w"] || keyDownMap["ArrowUp"]) {
          mesh.moveWithCollisions(mesh.forward.scaleInPlace(Currentspeed));    
          keydown = true; 
  
          } 
          if (keyDownMap["a"] || keyDownMap["ArrowLeft"]) {
          mesh.rotate(Vector3.Up(), -rotationSpeed);
          keydown = true; 
  
          } 
          if (keyDownMap["s"] || keyDownMap["ArrowDown"]) {
          mesh.moveWithCollisions(mesh.forward.scaleInPlace(-speedBackward));
          keydown = true; 
  
          } 
          if (keyDownMap["d"] || keyDownMap["ArrowRight"]) {
          mesh.rotate(Vector3.Up(), rotationSpeed);
          keydown = true; 
  
          }
          if(keyDownMap["e"] )
          {
            shiftdown = true;
          }
          else
          {
            shiftdown = false;
          }
          if (keydown && !isPlaying && !shiftdown){
            if(!animating){
              idleAnim = scene.stopAnimation(skeleton);
              walkAnim = scene.beginWeightedAnimation(skeleton, walkRange.from, walkRange.to, 1.0, true);
              animating = true;
            }
            if(animating){
              isPlaying = true;
            }
          }
          if(isPlaying && shiftdown && !isRunning){
            Currentspeed = Sprintspeed;
            walkAnim = scene.stopAnimation(skeleton);
            runAnim = scene.beginWeightedAnimation(skeleton, runRange.from, runRange.to, 1.0, true);
            isRunning = true;
          }
          if(isRunning && !shiftdown){
            Currentspeed = WalkSpeed;
            runAnim = scene.stopAnimation(skeleton);
            isRunning = false;
            if(keydown){
              walkAnim = scene.beginWeightedAnimation(skeleton, walkRange.from, walkRange.to, 1.0, true);;
            }
          }
  
          if(animating && !keydown){
            walkAnim = scene.stopAnimation(skeleton);
            idleAnim = scene.beginWeightedAnimation(skeleton, idleRange.from, idleRange.to, 1.0, true);
            animating = false;
            isPlaying = false;
          } 
              
  
        })
  
        //physics collision
        item = mesh; 
        let playerAggregate = new PhysicsAggregate(item, PhysicsShapeType.CAPSULE, { mass: 0}, scene);
        playerAggregate.body.disablePreStep = false;
  
        child.parent = item;
      });
      item.receiveShadows = true;
      return item;
    }

      
  function createBox(scene: Scene, player: any,  x: number, y: number, z: number){
    let box = MeshBuilder.CreateBox("box",{size: 1}, scene);
    box.position.y = 2; 

    let boxMat = new StandardMaterial("Head", scene);
    boxMat.alpha = 0;
    box.material = boxMat;
    return box; 
   } 
```

This is code is used to set up the Character model that player controls, and gives it physics, and toggles animations depending on player inputs
and sets speed variables for the player to switch between when walking or sprinting.

The box is used as a proxy collider due to issues with collisions, the box is parented to the player so that it follows it around and its alpha is set to 0
so that it is invisible.

## Pin Creation Code
```typescript
clonePin(that.scene, that.ground, that.box, that.Directionallight);

    function clonePin(scene: Scene, ground: Mesh, player: Mesh, light: DirectionalLight)
    {
     var time = 0;
     var RoundedTime;
     var shadowGenerator = new ShadowGenerator(1024, light);
     
     scene.onBeforeRenderObservable.add((thisScene, state) => {
       if (!thisScene.deltaTime) return;
       time += (thisScene.deltaTime / 1000);
       RoundedTime = Math.round(time);
       if (RoundedTime >= 1){
         time = 0;
         
         let pin: Mesh = createPinner(scene, ground, player ,Math.random() * (20 - -20) + -20, 15, Math.random() * (20 - -20) + -20);
         shadowGenerator.addShadowCaster(pin);
 
         let pin2: Mesh = createPinner(scene, ground, player ,Math.random() * (20 - -20) + -20, 15, Math.random() * (20 - -20) + -20);
         shadowGenerator.addShadowCaster(pin2);
 
         let pin3: Mesh = createPinner(scene, ground, player ,Math.random() * (20 - -20) + -20, 15, Math.random() * (20 - -20) + -20);
         shadowGenerator.addShadowCaster(pin3);
 
         let pin4: Mesh = createPinner(scene, ground, player ,Math.random() * (20 - -20) + -20, 15, Math.random() * (20 - -20) + -20);
         shadowGenerator.addShadowCaster(pin4);
 
         let pin5: Mesh = createPinner(scene, ground, player ,Math.random() * (20 - -20) + -20, 15, Math.random() * (20 - -20) + -20);
         shadowGenerator.addShadowCaster(pin5);
       }
       });
    }


    let hitCounter: number = 0;

    function createPinner(scene: Scene, collider: Mesh, player: Mesh, px: number, py: number, pz: number)
    {
      let Pinner = MeshBuilder.CreateCylinder("Pinner",{ diameterBottom: 0, diameterTop: 1, height: 2 }, scene);
      Pinner.position = new Vector3(px,py,pz);
  
      let PinMat = new StandardMaterial("Pin", scene);
      PinMat.diffuseColor = new Color3(255, 0, 0);
      PinMat.specularColor = new Color3(255, 0, 0);
      Pinner.material = PinMat;
 
 
      const PinAggregate = new PhysicsAggregate(Pinner, PhysicsShapeType.BOX, { mass: 1 }, scene);
      
 
      scene.onBeforeRenderObservable.add((thisScene, state) => {
       if (!thisScene.deltaTime) return;
       if (Pinner.intersectsMesh(collider) && !Pinner.isDisposed()) {
         Pinner.dispose();
        } 
        if (Pinner.intersectsMesh(player) && !Pinner.isDisposed()) {
         Pinner.dispose();
         console.log("player hit");
         hitCounter += 1;
        } 
       });
 
       Pinner.receiveShadows = true;
      return Pinner;
    }
  
```
This code is used to create five pins in random spots around the scene at one second intervals, when each pin is created it gets added to the list of shadow casters
so that each individual pin will cast their own shadow, once created the pins will fall from their spawn point until they hit either the ground or the player. When
the pin collides with either of these the pin will be disposed but if it hits the player the UI variable 'hitCounter' gets incremented. 
Each pin also checks if it has been disposed already before triggering the collision code to avoid any issues of lingering colliders.

## Game UI Code
```typescript

function createText(scene: Scene, s:string, c: string, advtex) {
    let Timertext = new GUI.TextBlock();
    Timertext.text = "0";
    Timertext.color = c;
    Timertext.fontSize = s;
    Timertext.fontWeight = "bold";
    Timertext.left = -450;
    Timertext.top = -250;
    advtex.addControl(Timertext);

    let Hittext = new GUI.TextBlock();
    Hittext.text = "0";
    Hittext.color = c;
    Hittext.fontSize = s;
    Hittext.fontWeight = "bold";
    Hittext.left = -500;
    Hittext.top = -200;
    advtex.addControl(Hittext);
    
    hitCounter = 0;
    var time = 0;
    let sceneSwitched: boolean = false;
    scene.onBeforeRenderObservable.add((thisScene, state) => {
      if (!thisScene.deltaTime) return;
      time += (thisScene.deltaTime / 1000);
      if(time >= 60 && !sceneSwitched){
        sceneSwitched = true;
        setSceneIndex(0); 
        ResetScenes(0);
        
        advtex.removeControl(Timertext); 
        advtex.removeControl(Hittext); 
      }
      Timertext.text = "Time survived: " + String(Math.round(time));
      Hittext.text = "Hit " + hitCounter + " times."
  });
    return Timertext;
    }

```
This code controls both of the game UI aspects, the timer and the hitcounter. The timer gets the deltaTime divided by a thousand to turn it into seconds and rounds the number
so it can be displayed properly for the player, if the timer is over 60 the scene switches back to the main menu and resets it so that the menus UI is active again. 

The code also controls the hit counter for the player which gets incremented each time a pin collides with the player, this number is then displayed.

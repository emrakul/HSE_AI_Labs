### Project creation

Create Third Person C++ project

### Pickup actor creation

In this section we will create basic class to represent pickups in our game.

Steps to create new actor:
* Create new C++ class called Pickup that is based on standard class Actor
* Open the code editor to implement the class
* Note UCLASS and GENERATED_BODY macros - they allow interaction between Code and UE4 Editor

Add boolean flat that signifies whether pickup is active

```c++
	// Is true if pickup is active and false otherwise.
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Pickup)
	bool bIsActive;
```

UPROPERTY allows us to use the field inside game editor and in blueprints:
* EditAnywhere - Indicates that this property can be edited via property windows, archetypes and instances within the Editor.
* BlueprintReadWrite - This property can be read or written from a blueprint.

Add sphere component that will handle collisions

```c++
	// Component for handling collisions.
	UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category = Pickup)
	USphereComponent* BaseCollisionComponent;
```

* BlueprintReadOnly - This property can be read by blueprints, but not modified.

Add static mesh component that will represent pickup in real world

```c++
	// Component that represents pickup in the real world.
	UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category = Pickup)
	UStaticMeshComponent* PickupMesh;
```

* VisibleDefaultsOnly - Indicates that this property is only visible in property windows for archetypes, and cannot be edited.

Declare function that will be called on pick up
```c++
	// Called when pickup is collected.
	UFUNCTION(BlueprintNativeEvent)
	void OnPickedUp();
```

* BlueprintNativeEvent - This function is designed to be overridden by a Blueprint, but also has a native implementation. Provide a body named [FunctionName]_Implementation instead of [FunctionName]; the autogenerated code will include a thunk that calls the implementation method when necessary.

Initialize pickup fields:
* Pickup should be active by default
* Create collision component and set the root component to it
* Create pickup mesh component, turn on physics for it, attach it to root component

```c++
	// Sets default values
	APickup::APickup()
	{
		// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
		PrimaryActorTick.bCanEverTick = true;

		// Pickup is active by default.
		bIsActive = true;

		// Create collision component.
		BaseCollisionComponent = CreateDefaultSubobject<USphereComponent>(TEXT("RootComponent"));

		// Set collision component as a root component.
		RootComponent = BaseCollisionComponent;

		// Create pickup mesh component.
		PickupMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("PickupMesh"));

		// Turn on physics (e.g. gravity, collisions).
		PickupMesh->SetSimulatePhysics(true);

		// Attach mesh to root component.
		PickupMesh->SetupAttachment(RootComponent);
	}
```

Implement OnPickedUp function

```cpp
	void APickup::OnPickedUp_Implementation()
	{
		// There is no default behavior for Pickup base class.
	}
```

Note the implementation suffix - it is a base implementation of the method that can be extended in blueprints.

### Battery class creation

In this section we will create battery pickup class that will represent powerup that can be collected by player to obtain energy.

Start by creating new C++ class BatteryPickup based on Pickup.

Implicitly declare public constructor to be able to change its implementation
```c++
	public:
		ABatteryPickup();
```

Create PowerLevel field to represent battery power

```c++
	// The ammount of power stored in the battery.
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Power)
	float PoweLevel;
```

Override base class OnPickedUp method

```c++
	// Override base class method implementation to customize the battery behavior.
	void OnPickedUp_Implementation() override;
```

Initialize battery power level
```c++
	ABatteryPickup::ABatteryPickup()
	{
		// Set default power level for a battery.
		PowerLevel = 150.0f;
	}
```

Override pickup implementation to destroy the battery after pickup
```c++
	void ABatteryPickup::OnPickedUp_Implementation()
	{
		// Call parent implementation.
		Super::OnPickedUp_Implementation();
		// Destroy the battery after pickup.
		Destroy();
	}
```

### Add a controllable character to the project

The character is already available in ThirdPerson Tutorial.
All we need to do is to migrate the content and the input mappings to our project.

To do this we load the ThirdPerson project, migrate ThirdPersonBP and Mannequin assets, export input bindings from settings.
Then we load Lesson_1 project back. Note that assets are already migrated. We then export input bindings settings and open Example Level from the migrated assets.
Now everything should work fine and we can copy the character to our original map.

More information about migration: https://docs.unrealengine.com/latest/INT/Engine/Content/Browser/UserGuide/Migrate/index.html

### Empowering the character

In this section we will implement character pickup collection and implement battery effects

First we start by adding a mapping for collecting a pickup. For this go to Edit -> Project settings -> Engine -> Input -> Bindings.
There you can see two types of mappings:
* Action Mapping - used for discrete events (e.g jumping)
* Axis Mapping - used for continous events (e.g. movement, camera direction)

Add new action mapping called CollectPickups and bind it to E key.

Add code to represent character power level.

Implement pickups collection.

### Creating the batteries representation

Create new blueprint BP_BatteryPickup

Set static mesh to represent it. Use Rock for simplicity, later replace it with battery from ExampleContent project.
Change collision settings to overlap with World.

### Implementing spawn volume

In this section we will create a volume that will spawn batteries at random locations of the map.

### Implement game modes

In this section we will implement game modes: game will end when the player reaches zero energy level.
At this point spawning will stop and player camera will be freezed.

### Implement HUD

In this section we will implement text interface to display current power lowel and also show Game Over message in the end of the game.
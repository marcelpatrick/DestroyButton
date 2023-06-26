# DestroyButton

Objective: Have a button on the screen that destroys an object in game if pressed

## Visual Summary: 

![image](https://user-images.githubusercontent.com/12215115/234550311-901ebeca-efb1-44b3-8d33-e41977fe22a8.png)

## Ingridients:

- Create an AActor C++ class MyActor and a Blueprint based on that
  - in the BP include a mesh and place it into the world (this will be the object to be destroyed)
  - in project settings, create a tag "ExplodeActor"
  - in the actor BP, create a new tag "ExplodeActor"
- Create a PlayerController C++ class MyPlayerController and a Blueprint based on that
- Create a WidgetBlueprint: Right click > User Interface > Widget Blueprint
- Create a Blueprint based on the game's GameModeBase
- Import an emitter to the project
  
## Preparation:

- in [projectname].build.cs:
  - Include "UMG" inside PublicDependenciesModuleNames
  - Uncomment where it sayas "incomment if using Slate UI"
    
- Project Settings > Maps and Modes
  - Default GameMode: BP_DestroyButtonGameModeBase
  - Default Pawn Class: Default Pawn
  - Player Controller Class: BP_MyPlayerController]
    
## COOKING:

### MyActor class:
- In the Actor Blueprint, in the Event Graph, OnEventDestroyed Spawn an emitter at the location where the mesh was destroyed:

![image](https://user-images.githubusercontent.com/12215115/234263624-6e413c4c-4e5d-43b2-9ff5-fadafe775bd9.png)

### GameModeBase Class:
- In the header file,
- Define BeginPlay()
- declare a variable to be a reference to the class type UUserWidget and expose it to the GameModeBase Blueprint so that we can associate it to our widget. 
  
```cpp
#include "Blueprint/UserWidget.h"

#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "ExplodeButtonGameModeBase.generated.h"

UCLASS()
class EXPLODEBUTTON_API AExplodeButtonGameModeBase : public AGameModeBase
{
	GENERATED_BODY()

public:
	void BeginPlay() override;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "MyWidgetClassType") 
	TSubclassOf<UUserWidget> MyUserWidgetClassType;
	
};
```

- In the cpp file,
- Define BeginPlay() and inherit its properties from the parent function with Super::
- create a widget and add it to the viewport
  
```cpp
#include "ExplodeButtonGameModeBase.h"

void AExplodeButtonGameModeBase::BeginPlay()
{
    Super::BeginPlay();

    if (MyUserWidgetClassType != nullptr)
    {
        UUserWidget* MyWidget = CreateWidget(GetWorld(), MyUserWidgetClassType);

        if (MyWidget != nullptr)
        {
            MyWidget->AddToViewport();
        }
    }
}
```

- In BP_ExplodeButtonGameModeBase, in the My User Widget Class Type field, select my BP_WidgetBlueprint to make sure the GameMode creates a widget based on the blueprint I am going to customize.

### MyPlayerController Class:
- In MyPlayerController header file, declare BeginPlay(),
- declare an object of type MyActor and a Array of Actors
- declare a DestroyActor() function and make it callable from the blueprints - so that we can call it from the actor blueprint to destroy it.
  
```cpp
#include "MyActor.h"

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "MyPlayerController.generated.h"

UCLASS()
class EXPLODEBUTTON_API AMyPlayerController : public APlayerController
{
	GENERATED_BODY()

public:
	virtual void BeginPlay() override;

	UFUNCTION(BlueprintCallable)

	void DestroyActor();

	TArray<AActor>* MyActorArray;

private:
	AMyActor* MyActor;
	
};
```

- In MyPlayerController cpp, 
  - OnBeginPlay(),
    - set input mode to be for both game and UI (to allow the player to both control the pawn and click on the screen),
    - set cursor to be visible,
    - get all actors with tag "ExplodeActor" and store the first one in the array of actors,
    - Save the first actor of the array into an AActor variable
    - define DestroyActor() using the actor object and calling the Destroy() function on it
```cpp
#include "Kismet/GameplayStatics.h"
#include "Engine/World.h" 

#include "MyPlayerController.h"


void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    SetInputMode(FInputModeGameAndUI());

    bShowMouseCursor = true;

    UGameplayStatics::GetAllActorsWithTag(GetWorld(), "ExplodeActor", MyActorArray);

    if (MyActorArray.Num() > 0)
    {

        MyActor = Cast<AMyActor>(MyActorArray[0]);
    }
    else
    {
        UE_LOG(LogTemp, Warning, TEXT("MyActorArray = EMPTY !!!"))
    }
    
}

void AMyPlayerController::DestroyActor()
{
    UE_LOG(LogTemp, Warning, TEXT("ACTOR DESTROYED !!!"))

    MyActor->Destroy();
}
```

### MyWidget Class:
- In the widget Blueprint, include a button and customize it
- Add a OnClicked event.
  - On the Event graph get the player controller and call its function Destroy() OnClicked.

![image](https://user-images.githubusercontent.com/12215115/234550380-e3964928-49ef-45d0-9457-798d54eed947.png)



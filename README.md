# DestroyButton

Objective: Have a button on the screen that destroys an object in game if pressed

## Ingridients:

- Create an AActor C++ class MyActor and a Blueprint based on that
  - in the BP include a mesh and place it into the world (this will be the object to be destroyed)
- Create a PlayerController C++ class MyPlayerController and a Blueprint based on that
- Create a WidgetBlueprint: Right click > User Interface > Widget Blueprint
- Create a Blueprint based on the game's GameModeBase
- Import an emitter to the project
  
## Preparation:

![image](https://user-images.githubusercontent.com/12215115/234550311-901ebeca-efb1-44b3-8d33-e41977fe22a8.png)

- in [projectname].build.cs:
  - Include "UMG" inside PublicDependenciesModuleNames
  - Uncomment where it sayas "incomment if using Slate UI"
    
- Project Settings > Maps and Modes
  - Default GameMode: BP_DestroyButtonGameModeBase
  - Default Pawn Class: Default Pawn
  - Player Controller Class: BP_MyPlayerController]
    
## COOKING:

### MyActor class:
- In the Actor Blueprint, include a mesh to it and in the Event Graph, OnEventDestroyed Spawn an emitter at the location where the mesh was destroyed:

![image](https://user-images.githubusercontent.com/12215115/234263624-6e413c4c-4e5d-43b2-9ff5-fadafe775bd9.png)

### GameModeBase Class:
- In the header file, declare a reference to a class type of type UUserWidget and expose it to the GameModeBase Blueprint
```cpp
#include "Blueprint/UserWidget.h"

#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "DontDestroyMeGameModeBase.generated.h"

/**
 * 
 */
UCLASS()
class DONTDESTROYME_API ADontDestroyMeGameModeBase : public AGameModeBase
{
	GENERATED_BODY()
	
public: 
	virtual void BeginPlay() override;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "My Widget")
	TSubclassOf<UUserWidget> MyWidget;

};
```

- In the cpp file, create a widget and add it to the viewport
```cpp
#include "DontDestroyMeGameModeBase.h"


void ADontDestroyMeGameModeBase::BeginPlay()
{
    Super::BeginPlay();

    if (MyWidget != nullptr)
    {
        //Create the widget in the world
        UUserWidget* MyCurrentWidget = CreateWidget<UUserWidget>(GetWorld(), MyWidget);

        if (MyCurrentWidget != nullptr)
        {
            //Add widget to the viewport
            MyCurrentWidget->AddToViewport();
        }   
    }
}
```

- In BP_DestroyButtonGameModeBase, select my MyWidgetBlueprint to make sure the GameMode create the widget blueprint I am going to customize.

### MyPlayerController Class:
- In MyPlayerController header file, declare an object of type MyActor and a Array of Actors and declare a DestroyActor() function and make it callable from the blueprints - so that we can call it from the actor blueprint to destroy it.
```cpp
#include "MyPawn.h"

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "MyPlayerController.generated.h"

/**
 * 
 */
UCLASS()
class DONTDESTROYME_API AMyPlayerController : public APlayerController
{
	GENERATED_BODY()

public:
	virtual void BeginPlay() override;

	UFUNCTION(BlueprintCallable)
	void DestroyWidget();

	TArray<AActor*> MyActors;

private:
	AMyPawn* MyPawn; 
};
```

- In MyPlayerController cpp, 
  - OnBeginPlay(), set input mode to be for both game and UI (to allow the player to both control the pawn and click on the screen), set cursor to be visible, get all actors of class AActor and store the first one in the array of actors, define DestroyActor() using the actor object and calling the Destroy() function on it
```cpp
#include "Kismet/GameplayStatics.h"

#include "Engine/World.h" 
#include "MyPlayerController.h"


void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    SetInputMode(FInputModeGameAndUI());

    bShowMouseCursor = true;

    UGameplayStatics::GetAllActorsOfClass(GetWorld(), AMyPawn::StaticClass(), MyActors);  

    MyPawn = Cast<AMyPawn>(MyActors[0]); 
}

void AMyPlayerController::DestroyWidget()
{
    MyPawn->Destroy();
}
```

### MyWidget Class:
- In the widget Blueprint, include a button and customize it
- Add a OnClicked event. On the Event graph get the player controller and call its function Destroy() OnClicked.

![image](https://user-images.githubusercontent.com/12215115/234550380-e3964928-49ef-45d0-9457-798d54eed947.png)



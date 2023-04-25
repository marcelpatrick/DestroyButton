# DestroyButton

Objective: Have a button on the screen that destroys an object in game if pressed

## Ingridients:

- Create an AActor C++ class MyActor and a Blueprint based on that
- Create a PlayerController C++ class MyPlayerController and a Blueprint based on that
- Create a WidgetBlueprint C++ class MyWidgetBlueprint and a Blueprint based on that 
- Create a Blueprint based on the game's GameModeBase

## Preparation:

- Project Settings > Maps and Modes
  - Default GameMode: BP_DestroyButtonGameModeBase
  - Default Pawn Class: Default Pawn
  - Player Controller Class: BP_MyPlayerController
  
### GameModeBase Class:
- In the header file, declare a class object of type UUserWidget and expose it to the GameModeBase Blueprint
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
### MyActor class:
- In the Actor Blueprint, include a mesh to it and in the Event Graph, OnEventDestroyed Spawn an emitter at the location where the mesh was destroyed:

![image](https://user-images.githubusercontent.com/12215115/234263624-6e413c4c-4e5d-43b2-9ff5-fadafe775bd9.png)

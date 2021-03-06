# Misuno_FSM
# A class-based FSM realisation. 

## Usage:

0) Write some custom States and StateTransitions

1) include namespace Misuno to use it's classes

2) Create a root FSM
```csharp
rootFSM = new FSM (gameObject, name + "_root");
```
3) Create some states and add them to FSM (second parameter of AddState shows that findTarget sould be set as initial)
```csharp
var findTarget = new StateFindNearestTarget(this, "find target", minSearchRadius);
rootFSM.AddState(findTarget, true);

moveToTargetState = new StateMoveToTarget(this, "move to target", 0.01f, moveSpeed, rotationSpeed, acceleration);
rootFSM.AddState(moveToTargetState);
```
4) Add some transitions between states (note, than every state could have several transitions)
```csharp
rootFSM.AddTransition(new TransitionFinished(findTarget, moveToTargetState));
rootFSM.AddTransition(new TransitionTargetDestroyed(moveToTargetState, findTarget, this));
rootFSM.AddTransition(new TransitionTimer(moveToTargetState, findTarget, 15f));
```
5) You could make some complex behaviours by using FSMs as States. 
```csharp
var complexState = new FSM (gameObject, "complex behaviour");
rootFSM.AddState(complexState);

// Add some states and transitions to complexState.

rootFSM.AddTransition( new ST_Finished (moveToTarget, complexState));
```

## Implementing the State

To create their own state one should inherit a new class from a **State** class. 

Methods to override are:
```csharp
virtual public void Enter ()
virtual public void Update ()
virtual public void Exit ()
```

### Example state

This state is included into the repositiory.

```csharp
using UnityEngine;

namespace Misuno
{
    public class StateMoveToPosition : State
    {
        public readonly Vector3 destination;
        public float rotationSpeed = 0.1f;
        public float speed = 10f;
        public GameObject host;

        public StateMoveToPosition (GameObject sender, string name, Vector3 destination, float speed = 10f, float rotationSpeed = 0.1f) :
            base (name)
        {
            host = sender;
            this.destination = destination;
            this.speed = speed;
            this.rotationSpeed = rotationSpeed;
        }

        public override void Update ()
        {
            if (!finished)
            {
                var distance = Vector3.Distance (host.transform.position, destination);
                if (distance > 1f)
                {
                    var direction = destination - host.transform.position;
                    direction.Normalize ();
                    host.transform.rotation = Quaternion.Slerp (host.transform.rotation, Quaternion.LookRotation (direction), rotationSpeed);
                    host.transform.Translate (Vector3.forward * Time.deltaTime * speed);
                }
                else
                {
                    finished = true;
                }
            }
        }
    }
}

```

## Implementing the StateTransition

To create their own state transition one should inherit a new class from a **StateTransition** class. 

Method to override:

```csharp
virtual public bool Check ()
```

### Example state transition

This state transaction is included into the repositiory.

```csharp
using UnityEngine;

namespace Misuno
{
    public class TransitionTimer: StateTransition
    {
        public readonly float Duration;
        float timer = -1f;

        public TransitionTimer (State from, State to, float duration) :
            base (from, to)
        { 
            Duration = duration;
        }

        override public bool Check ()
        {
            if (timer < 0f)
            {
                timer = Time.time + Duration;
            }

            if (Time.time >= timer)
            {
                timer = -1f;
                return true;
            }
            return false;
        }
    }
}
```

## Roadmap
- [ ] implement visualisation for debug purposes
- [ ] add more general states
- [ ] include periodic call on the level of base classes


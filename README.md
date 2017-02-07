# agent-runtime

runtime to create JavaScript controlled agents which can interact with an external turn based simulated environment

```bash
npm install @robotopia/agent-runtime
```

## Usage

A agent is a JavaScript program which runs in a sandboxed interpreter. Agents can interact with the outside environment
through user defined APIs. The runtime is executed step by step. Every agent has one turn per step, they are executed
in sequence. An agent script will be interrupted after it spend all its available action points per round. The execution will
then be continued in the next step of the runtime.

### Create runtime

```JavaScript
const Runtime = require('@robotopia/agent-runtime')
const runtime = new Runtime()

// use to update internal state of runtime if it changed
runtime.setState({
  ROBOT_1: { position: { x: 0, y: 0 } },
  ROBOT_2: { position: { x: 0, y: 0 } },  
  ROBOT_3: { position: { x: 0, y: 0 } }
})

// add action handler
//
// id : id of agent which triggered event
// name: name of the action
// data: payload of the action
runtime.onAction((id, name, data) => {
  // ...execute action
})
```

### Define an agent api

> The runtime uses [esper.js](https://github.com/codecombat/esper.js) to interpret the agent scrips. Unfortunately they
> don't support es6 fully yet. This means all `methods` and `globals` of the api have to be written in es5

```JavaScript

const robotApi = {
  
  namespace: 'robot',
  
  globals: {
    random: function () { return Math.random() }
  },
  
  actionsPerRound: 1,
  
  actions: {
    move: (x, y) => ({
      action: ['move', { x, y }],
      cost: 1
    }),
  },

  methods: {
    moveTo: function (x, y) {},
  },

  sensors: {
    getPosition: (state, id) => {
      return state[id].position
    }
  }
}

```

### Create an agent

```JavaScript
const code = `  
  while (true) {
    robot.move(random() * 10, random() * 10)
  }
`

runtime.createAgent({
  id: 'ROBOT_1', // unique id for the agent
  groupId: 'TEAM_1', // adds agent to a group which is relevant when triggering events
  api: robotApi, // api object which defines how the agent interact with its environment
  code // javascript code which defines the behaviour of the agent 
})    

```

### Delete an agent

```Javascript
runtime.destroyAgent('robot1')

```

### Using Events

Events interrupt the execution of the regular program flow. The event handler function is executed instead. Once the 
handler terminates the regular program flow is continued. An agent can only handle one event at a time. If an event is 
triggered while the agent is still executing the handler of an previous event the new event will be ignored.

```JavaScript
const code = `
  robot.__eventHandlers = {
    moveTo: function (x, y) {
      robot.moveTo(x, y)
    }
  }
`

runtime.createAgent({
  id: 'ROBOT_2',
  groupId: 'TEAM_1',
  api: robotApi,
  code
})


// events can be either triggered on a specific agent 
runtime.triggerEvent('moveTo', [10, 20], { id : 'ROBOT_2'})


// .. on a specific group of agents
runtime.triggerEvent('moveTo', [10, 20], { groupId : 'TEAM_1'})

// ... or on all agent
runtime.triggerEvent('moveTo', [10, 20])
```

### Using Modes

Modes are very similar to events. The difference is that modes don't interrupt the program flow if the agent is already
executing a different mode handler.

```JavaScript
const code = `
  robot.__modeHandlers = {
    patrol: function () {
      while (true) {
        robot.moveTo(0, 0)
        robot.moveTo(0, 10)
      }
    }
  }
`

runtime.createAgent({
  id: 'ROBOT_3',
  groupId: 'TEAM_1',
  api: robotApi,
  code
})

// works analogous to event.triggerEvent
runtime.switchMode('patrol', [], { id: 'ROBOT_3' })
```

## License 

MIT

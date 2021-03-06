---
layout: post
title: Angular NgRx
image: https://user-images.githubusercontent.com/18272237/54095339-33c48180-437d-11e9-8a0d-d0ae230be804.png
---
Implementing Reactive State for Angular into my Web Application. This blog post will be used for taking notes while
integrating State Management into my Web Application for better maintainability. `View(calls action)` >> `Reducer(handles action)` >> `Store(update)` >> `Effects(calls API)` >> `Store(update)` >> `Selector(handles data)` >> `Component(UI)`

Using this blog post as a `notepad` while implementing NgRx into my web application.

### Overview

`State` is an immutable data structure storing the current status fo different parts for your application. You can access the state through the store, you can think of the store as a master brain for you application. 

`Store` is an observable of State, which you can _subscribe_ to. The store is an observer of actions, which are informational payloads that describe state changes. For the store to update the State when data is received, actions must be dispatched to the store.

`Actions` are payloads of information to update the state by dispatching to the store, an actions has a type and an optional payload. **`Always write actions first to plan out the different use cases of the application, actions are the backbone of an NgRx application.`** Good actions can be read a year later and still be able to easily find where they are dispatched.

`Reducers` are pure functions that use actions along with the previous state to produce the new state.

`Selectors` are alos pure functions that are used to get simple and complex pieces of state. They act as queries to your store.

`Effects` are used to connect actions to side effects or external requests. Effects can also listen for actions and perform a side effect which that effect can dispatch a new action _if desired_. Effects helps reducers from containing too much implementation details, also keeps you state from being full of temporary clutter.

{: .mb-0}
### RxJS

{: .table .table-sm .table-bordered .table-striped .table-hover }
| Operator | Returns | Description |
| compose | Single | allows you create a custom operator |
| concat and concatWith | Observable | concatenates the items emitted by multiple Singles as Observable emissions |
| create | Single | create a Single from scratch by calling subscriber methods explicitly |
| delay | Single | move the emission of an item from a Single forward in time |
| doOnError | Single | returns a Single that also calls a method you specify when it calls onError |
| doOnSuccess | Single | returns a Single that also calls a method you specify when it calls onSuccess |
| error | Single | returns a Single that immediately notifies subscribers of an error |
| flatMap | Single | returns a Single that is the result of a function applied to an item emitted by a Single |
| flatMapObservable | Observable | returns an Observable that is the result of a function applied to an item emitted by a Single |
| from | Single | converts a Future into a Single |
| just | Single | returns a Single that emits a specified item |
| map | Single | returns a Single that emits the result of a function applied to the item emitted by the source Single |
| merge | Single | converts a Single that emits a second Single into a Single that emits the item emitted by the second Single |
| merge and mergeWith | Observable | merges the items emitted by multiple Singles as Observable emissions |
| observeOn | Single | instructs the Single to call the subscriber methods on a particular Scheduler |
| onErrorReturn | Single | converts a Single that makes an error notification into a Single that emits a specified item |
| switchMap | Oberservable | switch to a new observable |
| subscribeOn | Single | instructs the Single to operate on a particular Scheduler |
| timeout | Single | returns a Single that makes an error notification if the source Single does not emit a value in a specified time period |
| toSingle | Single | converts an Observable that emits a single item into a Single that emits that item |
| toObservable | Observable | converts a Single into an Observable that emits the item emitted by the Single and then completes |
| zip and zipWith | Single | returns a Single that emits an item that is the result of a function applied to items emitted by two or more other Singles |
{: .w-auto .mb-4 .text-center }

Functions used:
* pipe - 
* map - Apply projection with each value from source
* switchMap - Map to observable, complete previous inner observable emit values (Cancels the current subscription/request and can cause race condition USE for GET/cancellable requests)
* concat - runs subscriptions in order and is less performant ( USE for GET/POST/PUT requests when order is important)
* mergeMap - Map to observable, emit values | Runs subscriptions/requests in parallel (USER for PUT/POST/DELETE methods when order is not important)
* exhaustMap - Ignores all subsequent subscriptions/requests until it completes (USE for login when you do not want more requests until the initial one is complete)
* mapTo - One input, one output
* tap - 

##### Core Principles
- State is a single, immutable data structure.
- Components delegate responsibilities to side effects, which are handled in isolation.
- Type-safety is promoted throughout the architecture with reliance on TypeScript's compiler for program correctness.
- Actions and state are serializable to ensure state is predictably stored, rehydrated, and replayed.
- Promotes the use of functional programming when building reactive applications.
- Provide straightforward testing strategies for validation of functionality.

##### Packages
- Store - RxJS powered state management for Angular apps, inspired by Redux.
- Store Devtools - Instrumentation for @ngrx/store enabling time-travel debugging.
- Effects - Side effect model for @ngrx/store.
- Router Store - Bindings to connect the Angular Router to @ngrx/store.
- Entity - Entity State adapter for managing record collections.
- Schematics - Scaffolding library for Angular applications using NgRx libraries.

{: .mb-0}
`Process for creating/executing new Action for State:`
1. Write Action 
2. Define a reducer function
3. Update Component class with selector and methods to dispatch actions
4. Write an Effect


###### [Following Tutorial](https://ngrx.io/guide/store#tutorial):
- Goal: Show Users Tasks

> Steps to be taken:
> - Create `tasks.actions.ts` to describe tasks actions to add, update, and delete values
> - Create `tasks.reducer.ts` to handle changes in tasks based on the provided actions
> - Import `StoreModule` from `@ngrx/store` and the `tasks.reducer` file
> - Create `tasks.component.ts` to inject Store service for dispatching tasks actions, and use the select operator to
>   select data from the state


#### Writing actions - a few rules to write good actions
- `Upfront` - write actions before developing features to understand and gain a shared knowledge of the feature being implemented.
- `Divide` - categorize actions based on the event source.
- `Many` - actions are inexpensive to write, so the more actions you write, the better you express flows in your application.
- `Event-Driven` - capture events not commands as you are separating the description of an event and the handling of that event.
- `Descriptive` - provide context that are targeted to a unique event with more detailed information you can use to aid in debugging with the developer tools.


```typescript
// tasks.actions.ts
import { Action } from '@ngrx/store';
import { Task } from "../_models/Task";

export enum TaskActionTypes {
  FetchTasks = '[Tasks Component] Fetch Tasks',
  FetchTasksSuccess = '[Tasks Component] Fetch Tasks Success',
  CreateTask = '[Tasks Component] Create Task',
  UpdateTask = '[Tasks Component] Update Task',
  DeleteTask = '[Tasks Component] Delete Task',
}

export class FetchTasks implements Action {
  readonly type = TaskActionTypes.FetchTasks;
}

export class FetchTasksSuccess implements Action {
  readonly type = TaskActionTypes.FetchTasksSuccess;
  constructor(public payload: any) {}
}

export class CreateTask implements Action {
  readonly type = TaskActionTypes.CreateTask;
  constructor(public payload: any) { }
}

export class UpdateTask implements Action {
  readonly type = TaskActionTypes.UpdateTask;
  constructor(public payload: any) { }
}

export class DeleteTask implements Action {
  readonly type = TaskActionTypes.DeleteTask;
  constructor(public payload: any) { }
}


export type TasksAction = FetchTasks | FetchTasksSuccess | CreateTask | UpdateTask | DeleteTask;
```


#### Writing reducers - a few consistent parts
- An interface or type that defines the shape of the state.
- The arguments including the initial state or current state and the current action.
- The switch statement

```typescript
// tasks.reducer.ts
import { Action } from '@ngrx/store';
import { Task } from "../_models/Task";
import { UserTasks } from '../_models/userTasks';
import { TasksAction, TaskActionTypes } from './tasks.actions';

// Defining the state shape
// export interface State { tasks: Task[]; }

// Setting the initial state
export const initialState = [];

// Creating the reducer function
export function tasksReducer(state = initialState, action: TasksAction) {
  switch (action.type) {
    case TaskActionTypes.FetchTasks:
      return [ ...state ];

    case TaskActionTypes.FetchTasksSuccess:
      return [ ...state, action.payload ];

    case TaskActionTypes.CreateTask:
      return [ ...state, action.payload.task ];

    case TaskActionTypes.UpdateTask:
      return [ ...state, action.payload.task ];

    case TaskActionTypes.DeleteTask:
      return [ ...state, action.payload.task ];

    default:
      return state;
  }
}
```

#### Writing Effects to listen for events and perform tasks
> Effects are injectable service classes with distinct parts:
> - An injectable Actions service that provides an observable stream of all actions dispatched after the latest state has been reduced.
> - Observable streams are decorated with metadata using the Effect decorator. The metadata is used to register the streams that are subscribed to the store. Any action returned from the effect stream is then dispatched back to the Store.
> - Actions are filtered using a pipeable ofType operator. The ofType operator takes one more action types as arguments to filter on which actions to act upon.
> - Effects are subscribed to the Store observable.
> - Services are injected into effects to interact with external APIs and handle streams.

```typescript
// tasks.effects.ts
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { EMPTY } from 'rxjs';
import {catchError, map, mergeMap} from 'rxjs/operators';
import {AuthService} from "../@theme/auth/auth.service";
import {TaskActionTypes, TasksAction} from "./tasks.actions";

@Injectable()
export class TasksEffects {

  @Effect()
  loadTask$ = this.actions$
    .pipe(
      ofType(TaskActionTypes.FetchTasks),
      mergeMap(() => this.authService.getTasks()
        .pipe(
          map(tasks => ({ type: TaskActionTypes.FetchTasksSuccess, payload: tasks })),
          catchError(() => EMPTY)
        ))
      );

  constructor(
    private actions$: Actions,
    private authService: AuthService
  ) {}
}
```



#### Import StoreModule, EffectsModule, tasks.effects and tasks.reducer files
```typescript
// src/app/app/module.ts
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { TasksEffects } from './ngrx/tasks.effects';
import { tasksReducer } from './tasks.reducer';
```
Add the StoreModule.forRoot function in the imports array of your AppModule with an object containing the count and
the tasksReducer that manages the state of the tasks. The StoreModule.forRoot() method registers the global providers
needed to access the Store throughout your application.
```typescript
// src/app/app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
 
import { AppComponent } from './app.component';
 
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { TasksEffects } from './ngrx/tasks.effects';
import { tasksReducer } from './tasks.reducer';
 
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    StoreModule.forRoot({ tasks: tasksReducer }),
    EffectsModule.forRoot([TasksEffects]),
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```


#### Inject Store service into component to dispatch actions
```typescript
// tasks.component.ts
import { Component } from '@angular/core';
import { Store, select } from '@ngrx/store';
import { Observable } from 'rxjs';
import { CreateTask, DeleteTask, FetchTasks ,UpdateTask } from '../tasks.actions';
import { Task } from '../models/tasks';
 
@Component({
  selector: 'anon-tasks',
  templateUrl: './tasks.component.html',
  styleUrls: ['./tasks.component.css'],
})
export class MyCounterComponent {
  task$: Observable<Task[]>;
 
  constructor(private store: Store<{ tasks: Task[] }>) {
    this.task$ = store.pipe(select('tasks'));
  }
 
  fetchTasks() {
      this.store.dispatch(new FetchTasks());
  }
 
  createTask(task: Task) {
    this.store.dispatch(new CreateTask(task));
  }
 
  updateTask(task: Task) {
    this.store.dispatch(new UpdateTask(task));
  }
  
  deleteTask(task_ident: Task.ident) {
    this.store.dispatch(new DeleteTask(task_ident));
  }
  
}
```

#### Add HTML into Component to user Functions
Not finished building the UI for the Task form
This on the ToDo List, ironic huh? Wasn't on purpose..
```html
<!-- Here will go the HTML Tasks Component to use Fetch, Create, Update, and Delete functions -->
<nb-card class='w-100' *ngIf="task$ | async as tasks" >
    <nb-card-header> Tasks: </nb-card-header>
    
    <nb-card-body>
        <nb-list>
          <nb-list-item *ngFor="let task of tasks"> {{ task | json }} </nb-list-item>
        </nb-list>
    </nb-card-body>
    
    <nb-card-footer>  </nb-card-footer>
</nb-card>
```

Add the Tasks component to the AppComponent template to be user
```html
<anon-tasks></anon-tasks>
```


***
[Angular NgRx Docs](https://ngrx.io/docs)



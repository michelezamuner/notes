# Storybook

Storybook is a tool that aids the development of Web components for the React, Vue and Angular Web frameworks. The main idea of Storybook is that Web components should be designed, developed and tested in isolation, meaning outside and independently from any application, in order to maximize development speed and reusability.

## Single component

Storybook is best used while following the Component-Driven Development methodology, that suggests that the development of our application's UI should start from the single components first, and then only in the end wire up all components together in the final pages, or screens.

For each component we are developing, we need at least two files: one is the Angular component itself, and the other is a stories file. The Angular component might look something like this:

```typescript
// src/app/components/task.component.ts
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-task',
  template: `
    <div class="list-item">
      <input type="text" [value]="task.title" readonly="true" />
    </div>
  `,
})
export class TaskComponent implements OnInit {
  title: string;
  @Input() task: any;

  @Output() onPinTask: EventEmitter<any> = new EventEmitter();

  @Output() onArchiveTask: EventEmitter<any> = new EventEmitter();

  constructor();

  ngOnInit() {}
}
```

The stories file is used by Storybook to handle our new component:

```typescript
// src/app/components/task.stories.ts
import { action } from '@storybook/addon-actions';
import { TaskComponent } from './task.component';
export default {
  title: 'Task',
  excludeStories: /.*Data$/,
}

export const actionsData = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

export const taskData = {
  id: '1',
  title: 'Test Task',
  state: 'TASK_INBOX',
  updated_at: new Date(2019, 0, 1, 9, 0),
};

// default task state
export const Default = () => {
  component: TaskComponent,
  props: {
    task: taskData,
    onPinTask: actionsData.onPinTask,
    onArchiveTask: actionsData.onArchiveTask,
  },
};

// pinned task state
export const Pinned = () => ({
  component: TaskComponent,
  props: {
    task: {
      ...taskData,
      state: 'TASK_PINNED',
    },
    onPinTask: actionsData.onPinTask,
    onArchiveTask: actionsData.onArchiveTask,
  },
});

// archived task state
export const Archived = () => ({
  component: TaskComponent,
  props: {
    task: {
      ...taskData,
      state: 'TASK_ARCHIVED',
    },
    onPinTask: actionsData.onPinTask,
    onArchiveTask: actionsData.onArchiveTask,
  },
});
```

Here we've defined three different stories: `Default`, `Pinned` and `Archived`. Each story is a permutation of our component, which in this case means a different state a component can be in. Stories are automatically picked up and rendered by Storybook from our `export`s, which is why we need to explicitly add `excludeStories` in order for other exported objects to not be regarded as stories.

The definition of a story is a function that returns a component class with the properties it has in a given state. In addition to the component's properties, we can also pass actions that might be performed on the component in that state. To build an action, we use the `action` function, that will provide a control in the Storybook UI for us to use to send a specific event to our component.

After a stories file is created, we need to add it to Storybook configuration, in order for the stories to be picked up:

```javascript
module.exports = {
  stories: ['../src/app/components/**/*.stories.ts'],
  addons: ['@storybook/addon-actions', '@storybook/addon-links', '@storybook/addon-notes'],
};
```

Once we set up Storybook for our empty component, we can start actually implementing it:

```typescript
// src/app/components/task.component.ts

import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
import { Task } from '../models/task.model';

@Component({
  selector: 'app-task',
  template: `
    <div class="list-item {{ task?.state }}">
      <label class="checkbox">
        <input
          type="checkbox"
          [defaultChecked]="task?.state === 'TASK_ARCHIVED'"
          disabled="true"
          name="checked"
        />
        <span class="checkbox-custom" (click)="onArchive(task.id)"></span>
      </label>
      <div class="title">
        <input type="text" [value]="task?.title" readonly="true" placeholder="Input title" />
      </div>
      <div class="actions">
        <a *ngIf="task?.state !== 'TASK_ARCHIVED'" (click)="onPin(task.id)">
          <span class="icon-star"></span>
        </a>
      </div>
    </div>
  `
})
export class TaskComponent implements OnInit {
  title: string;
  @Input() task: Task;

  @Output() onPinTask: EventEmitter<any> = new EventEmitter();

  @Output() onArchiveTask: EventEmitter<any> = new EventEmitter();

  constructor() {}

  ngOnInit() {}

  onPin(id: any) {
    this.onPinTask.emit(id);
  }

  onArchive(id: any) {
    this.onArchiveTask.emit(id);
  }
}
```

With this component definition, Storybook now displays the component in the three different states when we click on the three different labels "default", "pinned" and "archived" in the UI.

## Composite component

Once we built a component for a single task, we want to build another one for a list of tasks. This will not only display tasks as a list, but also reorder them to display pinned ones first, and remove archived ones.

```typescript
// src/app/components/task-list.component.ts

import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
import { Task } from '../models/task.model';

@Component({
  selector: 'app-task-list',
  template: `
    <div class="list-items">
      <div *ngIf="loading">loading</div>
      <div *ngIf="!loading && task.length === 0">empty</div>
      <app-task
        *ngFor="let task of tasks"
        [task]="task"
        (onArchiveTask)="onArchiveTask.emit($event)"
        (onPinTask)="onPinTask.emit($event)"
      >
      </app-task>
    </div>
  `,
})
export class TaskListComponent implements OnInit {
  @Input() tasks: Task[] = [];
  @Input() loading = false;

  @Output() onPinTask: EventEmitter<any> = new EventEmitter();
  @Output() onArchiveTask: EventEmitter<any> = new EventEmitter();

  constructor() {}

  ngOnInit() {}
}
```

```typescript
// src/app/components/task-list.stories.ts

import { moduleMetadata } from '@storybook/angular';
import { CommonModule } from '@angular/common';
import { TaskListComponent } from './task-list.component';
import { TaskComponent } from './task.component';
import { taskData, actionsData } from './task.stories';

export default {
  title: 'TaskList',
  excludeStories: /.*Data$/,
  decorators: [
    moduleMetadata({
      declarations: [TaskListComponent, TaskComponent],
      imports: [CommonModule],
    }),
  ],
};

export const defaultTasksData = [
  { ...taskData, id: '1', title: 'Task 1' },
  { ...taskData, id: '2', title: 'Task 2' },
  { ...taskData, id: '3', title: 'Task 3' }
  { ...taskData, id: '4', title: 'Task 4' }
  { ...taskData, id: '5', title: 'Task 5' }
  { ...taskData, id: '6', title: 'Task 6' }
];
export const withPinnedTaskData = [
  ...defaultTasksData.slice(0, 5),
  { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
];

// default TaskList state
export const Default = () => ({
  component: TaskListComponent,
  template: `
    <div style="padding: 3rem">
      <app-task-list [tasks]="tasks" (onPinTask)="onPinTask($event)" (onArchiveTask)="onArchiveTask($event)"></app-task-list>
    </div>
  `,
  props: {
    tasks: defaultTasksData,
    onPinTask: actionsData.onPinTask,
    onArchiveTask: actionsData.onArchiveTask,
  },
});

// TaskList with pinned tasks

```

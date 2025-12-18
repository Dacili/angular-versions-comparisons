# Angular versions comparisons 
---
# Older vs Angular 17 - Examples
## *ngFor vs @for
---

### ❌ Old way (Angular 2-16)  - `*ngFor`- Structural Directive
html  
```
<fc-option 
  *ngFor="let costCenter of unitOptions(); trackBy: trackById" 
  [value]="costCenter.id" 
  [disabled]="costCenter.disabled">
  {{ costCenter.name }}
</fc-option>
```
typescript  
```
trackById(index: number, item: any): number {
  return item.id;
}
```
### Track by
Before we could choose not to use trackBy, it's used for optimizing performances.     
**Angular has to know how to identify every element** in list if list changes (adding, deleting, reordering...).  
- **Without** track by: Angular **destroys and recreates all** DOM elements.  
- **With** track by: Angular knows exactly **what changed, only updates** that.

> **For Angular 17+ we must use track by.**

---

### ✅Angular 17+ `@for` - Built-in Control Flow
```
@for (costCenter of unitOptions(); track costCenter.id) {
  <fc-option [value]="costCenter.id" [disabled]="costCenter.disabled">
    {{ costCenter.name }}
  </fc-option>
}
```

---

## 📋 Kompletna lista novih template sintaksi (Angular 17+)

| Stari način | Novi način | 
|-------------|------------|
| `*ngIf="condition"` | `@if (condition) { }` | 
| `*ngFor="let item of items"` | `@for (item of items; track item.id) { }` | 
| `*ngSwitch` | `@switch (value) { @case (1) { } }` | 
| else in *ngIf | `@else { }` |
| else in *ngIf | `@empty { }` -> Prazna lista |

---

## 🔄 Sve nove sintakse u praksi

### 1️⃣ **`@if` / `@else` - zamenjuje `*ngIf`**

#### ❌ Old way
```html
<div *ngIf="editMode(); else addMode">
  <h4>Edit Your Facility</h4>
</div>
<ng-template #addMode>
  <h4>Add Your Facility</h4>
</ng-template>
```

#### ✅ Angular 17+
```html
@if (editMode()) {
  <h4>Edit Your Facility</h4>
} @else {
  <h4>Add Your Facility</h4>
}
```

---

### 2️⃣ **`@for` / `@empty` - zamenjuje `*ngFor`**

#### ❌ Old way
```html
<div *ngIf="costCenterOptions().length > 0; else noData">
  <fc-option 
    *ngFor="let cc of costCenterOptions(); trackBy: trackById" 
    [value]="cc.id">
    {{ cc.name }}
  </fc-option>
</div>
<ng-template #noData>
  <p>No cost centers available</p>
</ng-template>
```

#### ✅ Angular 17+
```html
@for (cc of costCenterOptions(); track cc.id) {
  <fc-option [value]="cc.id">{{ cc.name }}</fc-option>
} @empty {
  <p>No cost centers available</p>
}
```

---

### 3️⃣ **`@switch` / `@case` - zamenjuje `*ngSwitch`**

#### ❌ Old way
```html
<div [ngSwitch]="moduleStatuses()?.facilitySetup">
  <span *ngSwitchCase="'pending'">⏳ Pending</span>
  <span *ngSwitchCase="'in-progress'">🔄 In Progress</span>
  <span *ngSwitchCase="'completed'">✅ Completed</span>
  <span *ngSwitchDefault>❓ Unknown</span>
</div>
```

#### ✅ Angular 17+
```html
@switch (moduleStatuses()?.facilitySetup) {
  @case ('pending') {
    <span>⏳ Pending</span>
  }
  @case ('in-progress') {
    <span>🔄 In Progress</span>
  }
  @case ('completed') {
    <span>✅ Completed</span>
  }
  @default {
    <span>❓ Unknown</span>
  }
}
```

---

# Older vs Angular 19 way - Examples

## 1️⃣ **toSignal() - Observable → Signal**

### ❌ Old way (Observable + async pipe)
<img width="729" alt="image" src="https://github.com/user-attachments/assets/1a99da67-27a5-488e-b9fa-bef30153a443" />     
  
We use **$ at the end of Observable name**.   
We use the **same name in HTML** with async pipe.  
We **could use subscribe** as well in .ts.   

### ✅ Angular 19 (Signal)
<img width="798" alt="image" src="https://github.com/user-attachments/assets/35b048c8-e648-401b-aec7-95573718bde9" />  
  
use **toSignal**  
In **HTML** we're **calling it as it's function**.  

---
### Observable vs signal 

| Feature | Observable | toSignal() |
|---------|-----------|-----------|
| **Memory Management** | ❌ Manual unsubscribe | ✅ Auto cleanup |
| **Template Syntax** | `value$ \| async` | `value()` |
| **Async Operations** | ✅ Native support | ❌ Need Observable wrapper |
| **RxJS Operators** | ✅ Full access | ❌ Limited |
| **Performance** | 🟡 Good | 🟢 Better |
| **Learning Curve** | 🟡 Steep | 🟢 Easier |
| **Type Safety** | 🟡 Good | 🟢 Better |

## **Reading values in method .ts**

### ❌  Old way - observable
```typescript
onSave(btn: FcProgressButton): void {
  // Moraš subscribovati ili imati već subscribovan value
  let facility: SmartScheduleFacility | null = null;
  this.facility$.pipe(take(1)).subscribe(f => facility = f);
}
```

### ✅ Angular 19 - Signal
```typescript
facility = toSignal(this.store.select(selectFacility)); // can be undefined

// or if we want to make sure that the initial value is not undefined
 const facility = toSignal(
  this.store.select(selectFacility),
  { initialValue: null }  // 👈 Garantuje da nije undefined
);

onSave(btn: FcProgressButton): void {
  const existingFacility = this.facility(); // 👈 Direktan poziv, trenutna vrijednost!
}
```

---

## 2️⃣ **computed() - Derivovani state iz drugih signala**

### ❌  Old way (map operator)
```typescript
stepInfo$ = this.store.select(selectModuleStepInfo);

currentStep$ = this.stepInfo$.pipe(
  map(info => info.stepMap['facilitySetup'] ?? 1)
);
```
```html
<!-- Template -->
<span>Step {{ currentStep$ | async }}</span>
```

### ✅ Angular 19 (computed)
```typescript
stepInfo = toSignal(this.store.select(selectModuleStepInfo), {
  initialValue: { total: 0, stepMap: {} as Record<string, number> }
});

currentStep = computed(() => this.stepInfo().stepMap['facilitySetup'] ?? 1);
```
```html
<!-- Template -->
<span>Step {{ currentStep() }}</span>
```
### map vs computed 
| Feature | map() | computed() |
|---------|-------|-----------|
| **Memoization** | ❌ No | ✅ Yes |
| **Performance** | 🟡 Creates Observable | 🟢 Optimized |
| **Chaining** | ✅ Pipe operators | ❌ Signal-only |
| **Async Support** | ✅ Yes | ❌ No |
| **Template** | Need `async` | Direct call `()` |


---

## 3️⃣ **Kombinovanje više Observables → combineLatest**

### ❌ Old way (combineLatest)
```typescript
private formValues$ = this.form.valueChanges.pipe(startWith(this.form.value));
costCenterOptions$ = this.store.select(selectCostCenterOptions);

unitOptions$ = combineLatest([
  this.formValues$,
  this.costCenterOptions$
]).pipe(
  map(([values, options]) => 
    options.map(cc => ({
      ...cc,
      disabled: cc.id === values.departmentCostCenter || cc.id === values.positionCostCenter
    }))
  )
);
```
```html
<!-- Template -->
@for (costCenter of (unitOptions$ | async); track costCenter.id) {
  <fc-option [value]="costCenter.id" [disabled]="costCenter.disabled">
    {{ costCenter.name }}
  </fc-option>
}
```

### ✅ Angular 19 (computed with more signals)
```typescript
private _formValues = toSignal(this.form.valueChanges.pipe(startWith(this.form.value)), {
  initialValue: this.form.value
});
costCenterOptions = toSignal(this.store.select(selectCostCenterOptions), { initialValue: [] });

unitOptions = computed(() => {
  const values = this._formValues();
  return this.costCenterOptions().map(cc => ({
    ...cc,
    disabled: cc.id === values.departmentCostCenter || cc.id === values.positionCostCenter
  }));
});
```
```html
<!-- Template -->
@for (costCenter of unitOptions(); track costCenter.id) {
  <fc-option [value]="costCenter.id" [disabled]="costCenter.disabled">
    {{ costCenter.name }}
  </fc-option>
}
```

---

## 4️⃣ **effect() - Side effects (reacting on changes)**

### ❌  Old way (subscribe + takeUntil)
```typescript
private destroy$ = new Subject<void>();
facility$ = this.store.select(selectFacility);

constructor() {
  this.facility$.pipe(
    takeUntil(this.destroy$),
    filter(facility => !!facility)
  ).subscribe(facility => {
    this.form.patchValue({
      facilityName: facility.FacilityName ?? '',
      weekStartDay: facility.WeekStarts ?? null,
      unitCostCenter: facility.DivisionCostCenterLevel1ID ?? null,
      departmentCostCenter: facility.DivisionCostCenterLevel2ID ?? null,
      positionCostCenter: facility.DivisionCostCenterLevel3ID ?? null,
    });
    this.form.markAsPristine();
  });
}

ngOnDestroy(): void {
  this.destroy$.next();
  this.destroy$.complete();
}
```

### ✅ Angular 19 (effect)
```typescript
facility = toSignal(this.store.select(selectFacility));

constructor() {
  effect(() => {
    const facility = this.facility();
    if (facility) {
      this.form.patchValue({
        facilityName: facility.FacilityName ?? '',
        weekStartDay: facility.WeekStarts ?? null,
        unitCostCenter: facility.DivisionCostCenterLevel1ID ?? null,
        departmentCostCenter: facility.DivisionCostCenterLevel2ID ?? null,
        positionCostCenter: facility.DivisionCostCenterLevel3ID ?? null,
      });
      this.form.markAsPristine();
    }
  });
}

// ✅ Nema potrebe za ngOnDestroy - auto cleanup!
```
### subscribe vs effect
| Feature | subscribe() | effect() |
|---------|-----------|----------|
| **Cleanup** | ❌ Manual | ✅ Auto |
| **Re-execution** | ❌ Manual | ✅ Auto on dependency change |
| **Error Handling** | ✅ Built-in | ❌ Limited |
| **Cancellation** | ✅ unsubscribe() | ❌ No control |
| **Memory Leaks** | ⚠️ High risk | ✅ Safe |
---

## 5️⃣ **inject() - Dependency Injection**

### ❌  Old way (constructor injection)
```typescript
constructor(
  private store: Store,
  private actions: Actions,
  @Inject(NOTIFICATION_SERVICE) private notificationService: _NotificationService
) {
  super(FacilitySetupIcons, FacilitySetupActions);
}
```

### ✅ Angular 19  (inject funkcija)
```typescript
private readonly actions$ = inject(Actions);
// store se injectuje u FeatureBase parent klasi

constructor(@Inject(NOTIFICATION_SERVICE) private notificationService: _NotificationService) {
  super(FacilitySetupIcons, FacilitySetupActions);
}
```

---


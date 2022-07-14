First. we need to install angular material with `ng add @angular/material`. Then we import modules that we use `import {MatToolbarModule} from '@angular/material/toolbar';`

### How to add Dialog Material?

When we wanted to add a click event for open Dialog to the button, we added the `openDialog()` function to the ts file.

```tsx
constructor(private dialog: MatDialog) {}
  openDialog() {
    this.dialog.open(DialogComponent, {});
  }
```

When we click the button, the dialog component opens. You can see the relevant code block

`open(DialogComponent, {})`.

### How to add Form?

We need to import these two module,

```tsx
import { MatFormFieldModule } from "@angular/material/form-field";
import { MatInputModule } from "@angular/material/input";
```

otherwise, the form will not work correctly.

### How to add Date Input?

We need to import these two module.

```tsx
import { MatDatepickerModule } from "@angular/material/datepicker";
import { MatNativeDateModule } from "@angular/material/core";
```

### How to Save Form with Angular Material?

First, we need to create our form. We create a dialog component and add these codes.

```tsx
<h1 mat-dialog-title>Add Product Form</h1>
<div mat-dialog-content>
  <form [formGroup]="productForm">
    <mat-form-field appearance="outline">
      <mat-label>Product Name</mat-label>
      <input
        formControlName="productname"
        matInput
        placeholder="Name"
        autocomplete="off"
      />
    </mat-form-field>

    <label id="example-radio-group-label">Pick your product freshness</label>
    <mat-radio-group
      formControlName="freshness"
      aria-labelledby="example-radio-group-label"
      class="example-radio-group"
    >
      <mat-radio-button
        class="example-radio-button"
        *ngFor="let fresh of freshnessList"
        [value]="fresh"
      >
        {{ fresh }}
      </mat-radio-button>
    </mat-radio-group>

  </form>
  <div mat-dialog-action [align]="'end'">
    <button mat-raised-button color="warn" mat-dialog-close>Cancel</button>
    <button
      mat-raised-button
      color="primary"
      style="margin-left: 8px"
      (click)="addProduct()"
    >
      Save
    </button>
  </div>
</div>
```

I didn’t add all codes to avoid complexity. We should notice that all input elements should gather by the `<form></form>` tag and assign productForm to formGroup. “productForm” includes all inputs value. We will define productForm in dialog component.

```tsx
import { FormBuilder, FormGroup, Validators } from "@angular/forms";
@Component({
  selector: "app-dialog",
  templateUrl: "./dialog.component.html",
  styleUrls: ["./dialog.component.scss"],
})
export class DialogComponent implements OnInit {
  freshnessList = ["New Brand", "Second Hand", "Refurbished"];
  productForm!: FormGroup;

  constructor(private formBuilder: FormBuilder) {}

  ngOnInit(): void {
    this.productForm = this.formBuilder.group({
      productname: ["", Validators.required],
      category: ["", Validators.required],
      freshness: ["", Validators.required],
      price: ["", Validators.required],
      comment: ["", Validators.required],
      date: ["", Validators.required],
    });
  }
  addProduct() {
    console.log(this.productForm.value);
  }
}
```

We see the freshnessList above code. This is for radio button values. Also, we see that productForm that type FormGroup. I mentioned that a productForm contains input values in form. We create a constructor to use FormBuilder. FormBuilder contains methods that allow us to group input values into productForm. “Validators.required” means this value is required, it cannot be left blank.

### Create api Service

After create api service we add these codes.

```tsx
import { HttpClient } from "@angular/common/http";
import { Injectable } from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class ApiService {
  constructor(private http: HttpClient) {}

  postProduct(data: any) {
    return this.http.post<any>("http://localhost:3000/productList/", data);
  }

  getProduct() {
    return this.http.get<any>("http://localhost:3000/productList/");
  }

  putProduct(data: any, id: number) {
    return this.http.put<any>("http://localhost:3000/productList/" + id, data);
  }
  deleteProduct(id: number) {
    return this.http.delete<any>("http://localhost:3000/productList/" + id);
  }
}
```

Then, we will use this method in app.module with table material.

### How to use Table Material?

Table material seems more complicated than the other materials. We implement codes from Angular Materials site. Let me check these codes.

```tsx
<div class="container">
  <div style="margin-top: 10px">
    <mat-form-field appearance="standard">
      <mat-label>Filter</mat-label>
      <input
        matInput
        (keyup)="applyFilter($event)"
        placeholder="Ex. Mia"
        #input
      />
    </mat-form-field>

    <div class="mat-elevation-z8">
      <table mat-table [dataSource]="dataSource" matSort>
        <!-- Product Name Column -->
        <ng-container matColumnDef="productname">
          <th mat-header-cell *matHeaderCellDef mat-sort-header>Product</th>
          <td mat-cell *matCellDef="let row">{{ row.productname }}</td>
        </ng-container>

        <!-- Date Column -->
        <ng-container matColumnDef="date">
          <th mat-header-cell *matHeaderCellDef mat-sort-header>Date</th>
          <td mat-cell *matCellDef="let row">{{ row.date | date }}</td>
        </ng-container>

        <!-- Action Column -->
        <ng-container matColumnDef="action">
          <th mat-header-cell *matHeaderCellDef mat-sort-header>Action</th>
          <td mat-cell *matCellDef="let row">
            <button mat-icon-button color="primary" (click)="editProduct(row)">
              <mat-icon>edit</mat-icon>
            </button>
            <button
              mat-icon-button
              color="warn"
              (click)="deleteProduct(row.id)"
            >
              <mat-icon>delete</mat-icon>
            </button>
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>

        <!-- Row shown when there is no matching data. -->
        <tr class="mat-row" *matNoDataRow>
          <td class="mat-cell" colspan="4">
            No data matching the filter "{{ input.value }}"
          </td>
        </tr>
      </table>

      <mat-paginator
        [pageSizeOptions]="[5, 10, 25, 100]"
        aria-label="Select page of users"
      ></mat-paginator>
    </div>
  </div>
```

In app.component.ts file:

```tsx
displayedColumns: string[] = [
    'productname',
    'category',
    'price',
    'freshness',
    'comment',
    'date',
    'action',
  ];

  dataSource!: MatTableDataSource<any>;

  @ViewChild(MatPaginator) paginator!: MatPaginator;
  @ViewChild(MatSort) sort!: MatSort;

constructor(private dialog: MatDialog, private api: ApiService) {}
  ngOnInit(): void {
    this.getAllProduct();
  }

  openDialog() {
    this.dialog
      .open(DialogComponent, {})
      .afterClosed()
      .subscribe((val) => {
        if (val === 'save') {
          this.getAllProduct();
        }
      });
  }

  getAllProduct() {
    this.api.getProduct().subscribe({
      next: (res) => {
        this.dataSource = new MatTableDataSource(res);
        this.dataSource.paginator = this.paginator;
        this.dataSource.sort = this.sort;
      },
      error: () => {
        alert('Error while fetching the products ');
      },
    });
  }

  editProduct(row: any) {
    this.dialog
      .open(DialogComponent, {
        data: row,
      })
      .afterClosed()
      .subscribe((val) => {
        if (val === 'update') {
          this.getAllProduct();
        }
      });
  }

  deleteProduct(id: number) {
    this.api.deleteProduct(id).subscribe({
      next: (res) => {
        alert('Product deleted succesfully!');
        this.getAllProduct();
      },
      error: () => {
        alert('Error while deleting!');
      },
    });
  }

applyFilter(event: Event) {
    const filterValue = (event.target as HTMLInputElement).value;
    this.dataSource.filter = filterValue.trim().toLowerCase();

    if (this.dataSource.paginator) {
      this.dataSource.paginator.firstPage();
    }
  }
```

Let me explain the codes starting over.

```tsx
displayedColumns: string[] = [
    'productname',
    'category',
    'price',
    'freshness',
    'comment',
    'date',
    'action',
  ];
```

In this code, displayedColums shows us table materials columns. We get the data from api service for the value of every column.

`dataSource!: MatTableDataSource<any>` in this code we create a `dataSource` variable that type is `MatTableDataSource`. This means that, columns variables come from this dataSource. We fetch data to here.

```tsx
getAllProduct() {
    this.api.getProduct().subscribe({
      next: (res) => {
        this.dataSource = new MatTableDataSource(res);
        this.dataSource.paginator = this.paginator;
        this.dataSource.sort = this.sort;
      },
      error: () => {
        alert('Error while fetching the products ');
      },
    });
  }
```

```tsx
  openDialog() {
    this.dialog
      .open(DialogComponent, {})
      .afterClosed()
      .subscribe((val) => {
        if (val === 'save') {
          this.getAllProduct();
        }
      });
  }
```

We call openDialog() function when we clicked the add product button. This function opens our dialog component.

### `MAT_DIALOG_DATA`

Injection token that can be used to access the data that was passed in to a dialog.

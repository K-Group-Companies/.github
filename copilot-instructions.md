# GitHub Copilot Instructions for DNN Development

## Project Context

This is a DNN (DotNetNuke) 10.1.2 project using:
- ASP.NET MVC with Razor views (.cshtml)
- Bootstrap 5.3 for UI
- jQuery with jQuery UI datepicker
- DAL2 with PetaPoco for data access
- SQL Server

## File Structure

```
DesktopModules/
└── MVC/
    └── [ModuleName]/
        ├── Controllers/
        │   └── ItemController.cs
        ├── Models/
        │   └── ItemInfo.cs
        ├── Views/
        │   └── Item/
        │       ├── Index.cshtml
        │       └── Edit.cshtml
        ├── Components/
        │   └── ItemRepository.cs
        ├── Providers/
        │   └── DataProviders/
        │       └── SqlDataProvider/
        │           └── [version].SqlDataProvider
        ├── App_LocalResources/
        │   └── Item.resx
        ├── Scripts/
        │   └── module.js
        └── [ModuleName].dnn
```

## MVC Controller Pattern

```csharp
using DotNetNuke.Web.Mvc.Framework.Controllers;
using DotNetNuke.Web.Mvc.Framework.ActionFilters;
using System.Web.Mvc;

namespace CompanyName.Modules.ModuleName.Controllers
{
    [DnnHandleError]
    public class ItemController : DnnController
    {
        private readonly ItemRepository _repository;

        public ItemController()
        {
            _repository = new ItemRepository();
        }

        public ActionResult Index()
        {
            var items = _repository.GetItems(ModuleContext.ModuleId);
            return View(items);
        }

        public ActionResult Edit(int itemId = 0)
        {
            var item = itemId > 0 
                ? _repository.GetItem(itemId, ModuleContext.ModuleId)
                : new ItemInfo { ModuleId = ModuleContext.ModuleId };
            return View(item);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit(ItemInfo item)
        {
            if (ModelState.IsValid)
            {
                if (item.ItemId == 0)
                {
                    item.CreatedByUserId = User.UserID;
                    item.CreatedOnDate = DateTime.Now;
                    _repository.AddItem(item);
                }
                else
                {
                    item.LastModifiedByUserId = User.UserID;
                    item.LastModifiedOnDate = DateTime.Now;
                    _repository.UpdateItem(item);
                }
                return RedirectToDefaultRoute();
            }
            return View(item);
        }
    }
}
```

## Razor View Patterns

### View Directive
```cshtml
@inherits DotNetNuke.Web.Mvc.Framework.DnnWebViewPage<CompanyName.Modules.ModuleName.Models.ItemInfo>
@using DotNetNuke.Web.Mvc.Helpers
```

### Bootstrap 5 Form Field
```cshtml
<div class="mb-3 row">
    <label class="col-sm-3 col-form-label">Field Label</label>
    <div class="col-sm-9">
        @Html.TextBoxFor(m => m.FieldName, new { @class = "form-control" })
        @Html.ValidationMessageFor(m => m.FieldName, "", new { @class = "text-danger" })
    </div>
</div>
```

### Date Picker Field
```cshtml
<div class="mb-3 row">
    <label class="col-sm-3 col-form-label">Date Field <span class="text-danger">*</span></label>
    <div class="col-sm-9">
        <div class="input-group">
            @Html.TextBoxFor(m => m.DateField, "{0:MM/dd/yyyy}", new { @class = "form-control datepicker" })
            <span class="input-group-text"><i class="fa fa-calendar"></i></span>
        </div>
        @Html.ValidationMessageFor(m => m.DateField, "", new { @class = "text-danger" })
    </div>
</div>
```

### Complete Edit View
```cshtml
@inherits DotNetNuke.Web.Mvc.Framework.DnnWebViewPage<CompanyName.Modules.ModuleName.Models.ItemInfo>
@using DotNetNuke.Web.Mvc.Helpers

<div class="card">
    <div class="card-header">
        <h5 class="mb-0">@Dnn.LocalizeString("EditItem")</h5>
    </div>
    <div class="card-body">
        @using (Html.BeginForm("Edit", "Item", FormMethod.Post, new { @class = "form-horizontal" }))
        {
            @Html.AntiForgeryToken()
            @Html.HiddenFor(m => m.ItemId)
            @Html.HiddenFor(m => m.ModuleId)

            <div class="mb-3 row">
                <label class="col-sm-3 col-form-label">Name <span class="text-danger">*</span></label>
                <div class="col-sm-9">
                    @Html.TextBoxFor(m => m.Name, new { @class = "form-control" })
                    @Html.ValidationMessageFor(m => m.Name, "", new { @class = "text-danger" })
                </div>
            </div>

            <div class="row">
                <div class="col-sm-9 offset-sm-3">
                    <button type="submit" class="btn btn-primary">@Dnn.LocalizeString("Save")</button>
                    <a href="@Url.Action("Index", "Item")" class="btn btn-secondary">@Dnn.LocalizeString("Cancel")</a>
                </div>
            </div>
        }
    </div>
</div>
```

### Index View
```cshtml
@inherits DotNetNuke.Web.Mvc.Framework.DnnWebViewPage<IEnumerable<CompanyName.Modules.ModuleName.Models.ItemInfo>>
@using DotNetNuke.Web.Mvc.Helpers

<div class="mb-3">
    <a href="@Url.Action("Edit", "Item")" class="btn btn-primary">
        <i class="fa fa-plus"></i> @Dnn.LocalizeString("AddItem")
    </a>
</div>

<div class="table-responsive">
    <table class="table table-striped table-hover">
        <thead>
            <tr>
                <th>@Dnn.LocalizeString("Name")</th>
                <th>@Dnn.LocalizeString("Date")</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model)
            {
                <tr>
                    <td>@item.Name</td>
                    <td>@item.DateField?.ToString("MM/dd/yyyy")</td>
                    <td>
                        <a href="@Url.Action("Edit", "Item", new { itemId = item.ItemId })" class="btn btn-sm btn-outline-primary">
                            <i class="fa fa-edit"></i>
                        </a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

## Entity Class with DAL2

```csharp
using DotNetNuke.ComponentModel.DataAnnotations;
using System;
using System.ComponentModel.DataAnnotations;
using System.Web.Caching;

namespace CompanyName.Modules.ModuleName.Models
{
    [TableName("ModuleName_Items")]
    [PrimaryKey("ItemId", AutoIncrement = true)]
    [Cacheable("ModuleName_Items_", CacheItemPriority.Default, 20)]
    [Scope("ModuleId")]
    public class ItemInfo
    {
        public int ItemId { get; set; }
        public int ModuleId { get; set; }
        
        [Required(ErrorMessage = "Name is required")]
        [StringLength(100, ErrorMessage = "Name cannot exceed 100 characters")]
        public string Name { get; set; }
        
        [Required(ErrorMessage = "Installation date is required")]
        public DateTime? InstallationDate { get; set; }
        
        public DateTime? WarrantyExpiration { get; set; }
        
        public int CreatedByUserId { get; set; }
        public DateTime CreatedOnDate { get; set; }
        public int LastModifiedByUserId { get; set; }
        public DateTime LastModifiedOnDate { get; set; }
    }
}
```

## Repository Pattern

```csharp
using DotNetNuke.Data;
using DotNetNuke.Common.Utilities;
using System.Collections.Generic;
using System.Linq;

namespace CompanyName.Modules.ModuleName.Components
{
    public class ItemRepository
    {
        public IEnumerable<ItemInfo> GetItems(int moduleId)
        {
            using (var context = DataContext.Instance())
            {
                var repository = context.GetRepository<ItemInfo>();
                return repository.Get(moduleId);
            }
        }

        public ItemInfo GetItem(int itemId, int moduleId)
        {
            using (var context = DataContext.Instance())
            {
                var repository = context.GetRepository<ItemInfo>();
                return repository.GetById(itemId, moduleId);
            }
        }

        public void AddItem(ItemInfo item)
        {
            using (var context = DataContext.Instance())
            {
                var repository = context.GetRepository<ItemInfo>();
                repository.Insert(item);
            }
            DataCache.ClearCache("ModuleName_Items_");
        }

        public void UpdateItem(ItemInfo item)
        {
            using (var context = DataContext.Instance())
            {
                var repository = context.GetRepository<ItemInfo>();
                repository.Update(item);
            }
            DataCache.ClearCache("ModuleName_Items_");
        }

        public void DeleteItem(int itemId, int moduleId)
        {
            using (var context = DataContext.Instance())
            {
                var repository = context.GetRepository<ItemInfo>();
                repository.Delete("WHERE ItemId = @0 AND ModuleId = @1", itemId, moduleId);
            }
            DataCache.ClearCache("ModuleName_Items_");
        }
    }
}
```

## SQL Scripts

### Add Column Pattern
```sql
IF NOT EXISTS (SELECT * FROM sys.columns WHERE object_id = OBJECT_ID(N'{databaseOwner}[{objectQualifier}TableName]') AND name = 'ColumnName')
BEGIN
    ALTER TABLE {databaseOwner}[{objectQualifier}TableName]
    ADD [ColumnName] [datetime] NULL
END
GO
```

### Create Table Pattern
```sql
IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'{databaseOwner}[{objectQualifier}ModuleName_Items]') AND type in (N'U'))
BEGIN
    CREATE TABLE {databaseOwner}[{objectQualifier}ModuleName_Items](
        [ItemId] [int] IDENTITY(1,1) NOT NULL,
        [ModuleId] [int] NOT NULL,
        [Name] [nvarchar](100) NOT NULL,
        [InstallationDate] [datetime] NULL,
        [WarrantyExpiration] [datetime] NULL,
        [CreatedByUserId] [int] NOT NULL,
        [CreatedOnDate] [datetime] NOT NULL,
        [LastModifiedByUserId] [int] NOT NULL,
        [LastModifiedOnDate] [datetime] NOT NULL,
        CONSTRAINT [PK_{objectQualifier}ModuleName_Items] PRIMARY KEY CLUSTERED ([ItemId] ASC)
    )
END
GO
```

## Date Picker JavaScript

```javascript
$(document).ready(function() {
    $('.datepicker').datepicker({
        format: 'mm/dd/yyyy',
        autoclose: true,
        todayHighlight: true,
        clearBtn: true
    });
});
```

## DNN Helpers Reference

| Helper | Usage |
|--------|-------|
| `@Dnn.LocalizeString("Key")` | Get localized string from .resx |
| `@Dnn.ModuleContext.ModuleId` | Current module ID |
| `@Dnn.ModuleContext.TabId` | Current page/tab ID |
| `@Dnn.User.UserID` | Current user ID |
| `@Url.Action("Action", "Controller")` | Generate URL to action |
| `@Html.AntiForgeryToken()` | CSRF protection token |

## Common Namespaces

```csharp
// Controllers
using DotNetNuke.Web.Mvc.Framework.Controllers;
using DotNetNuke.Web.Mvc.Framework.ActionFilters;
using System.Web.Mvc;

// Data Access
using DotNetNuke.Data;
using DotNetNuke.Common.Utilities;
using DotNetNuke.ComponentModel.DataAnnotations;

// Validation
using System.ComponentModel.DataAnnotations;
```

## Things to Avoid

- Don't use inline SQL without parameterization
- Don't forget `[ValidateAntiForgeryToken]` on POST actions
- Don't forget `@Html.AntiForgeryToken()` in forms
- Don't use hard-coded table names without `{objectQualifier}`
- Don't store dates as strings in the database
- Don't skip null checks on optional date fields: use `?.ToString()` in views
- Don't inherit from standard MVC ViewPage; use `DnnWebViewPage<T>`
- Don't forget `@Html.HiddenFor()` for ItemId and ModuleId in edit forms
- Don't forget to clear cache after repository updates when using `[Cacheable]`

# MudBlazor Theme in ABP Blazor WebAssembly PART 4

## Introduction

In this part, I'll show how to create CRUD pages with [MudBlazor](https://www.mudblazor.com/) components.

## 11. Make Sure You Have Completed Previous Parts

You must complete the steps in [Part 1](https://github.com/yellow-dragon-cloud/AbpMudBlazor), [Part 2](https://github.com/yellow-dragon-cloud/AbpMudBlazor2), and [Part 3](https://github.com/yellow-dragon-cloud/AbpMudBlazor3) to continue.

## 12. Create CRUD Pages

Complete the __Web Application Development Tutorial__ in __ABP documentation__:
- [Web Application Development Tutorial - Part 1: Creating the Server Side](https://docs.abp.io/en/abp/latest/Tutorials/Part-1?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 2: The Book List Page](https://docs.abp.io/en/abp/latest/Tutorials/Part-2?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 3: Creating, Updating and Deleting Books](https://docs.abp.io/en/abp/latest/Tutorials/Part-3?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 4: Integration Tests](https://docs.abp.io/en/abp/latest/Tutorials/Part-4?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 5: Authorization](https://docs.abp.io/en/abp/latest/Tutorials/Part-5?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 6: Authors: Domain Layer](https://docs.abp.io/en/abp/latest/Tutorials/Part-6?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 7: Authors: Database Integration](https://docs.abp.io/en/abp/latest/Tutorials/Part-7?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 8: Authors: Application Layer](https://docs.abp.io/en/abp/latest/Tutorials/Part-8?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 9: Authors: User Interface](https://docs.abp.io/en/abp/latest/Tutorials/Part-9?UI=Blazor&DB=Mongo)
- [Web Application Development Tutorial - Part 10: Book to Author Relation](https://docs.abp.io/en/abp/latest/Tutorials/Part-10?UI=Blazor&DB=Mongo)

## 13. Update __Authors__ CRUD Page

Add `@using MudBlazor` to `Acme.BookStore.Blazor/_Imports.razor`

Update `Authors.razor` with the following content:

```razor
@page "/authors"
@using Acme.BookStore.Authors
@using Acme.BookStore.Localization
@using Volo.Abp.AspNetCore.Components.Web
@inherits BookStoreComponentBase
@inject IAuthorAppService AuthorAppService
@inject AbpBlazorMessageLocalizerHelper<BookStoreResource> LH

<MudCard Elevation="8">
    <MudCardHeader>
        <MudGrid>
            <MudItem>
                <MudText Typo="Typo.h5">@L["Authors"]</MudText>
            </MudItem>
            <MudSpacer />
            <MudItem>
                <MudButton Color="MudBlazor.Color.Primary"
                           Variant="Variant.Outlined"
                           Disabled="!CanCreateAuthor"
                           OnClick="OpenCreateAuthorModal">
                    @L["NewAuthor"]
                </MudButton>
            </MudItem>
        </MudGrid>
    </MudCardHeader>
    <MudCardContent>
        <MudDataGrid T="AuthorDto"
                     @ref="_authorList"
                     Striped="true"
                     ServerData="LoadServerData">
            <Columns>
                <MudBlazor.Column T="AuthorDto"
                                  Field="@nameof(AuthorDto.Id)"
                                  Title="@L["Actions"]">
                    <CellTemplate>
                        @if (CanEditAuthor)
                        {
                            <MudIconButton Icon="fas fa-edit" 
                                           OnClick="@((e) => {OpenEditAuthorModal(context);})"
                                           Size="MudBlazor.Size.Small" />
                        }
                        @if (CanDeleteAuthor)
                        {   
                            <MudIconButton Icon="fas fa-trash" 
                                           OnClick="@(async (e) => {await DeleteAuthorAsync(context);})"
                                           Size="MudBlazor.Size.Small" />
                        }
                    </CellTemplate>
                </MudBlazor.Column>
                <MudBlazor.Column T="AuthorDto"
                                  Field="@nameof(AuthorDto.Name)"
                                  Title="@L["Name"]" />
                <MudBlazor.Column T="AuthorDto"
                                  Field="@nameof(AuthorDto.BirthDate)"
                                  Title="@L["BirthDate"]">
                    <CellTemplate>
                        @context.BirthDate?.ToShortDateString()
                    </CellTemplate>
                </MudBlazor.Column>
            </Columns>
        </MudDataGrid>
    </MudCardContent>
</MudCard>

<MudDialog @bind-IsVisible="_createAuthorDialogVisible">
    <TitleContent>
        <MudText Typo="Typo.h6">@L["NewAuthor"]</MudText>
    </TitleContent>
    <DialogContent>
        <MudForm Model="@NewAuthor"
                 @ref="_createForm">
            <MudTextField @bind-Value="@NewAuthor.Name" 
                          Label=@L["Name"]
                          For=@(() => NewAuthor.Name) />
            <br />
            <MudDatePicker @bind-Date="@NewAuthor.BirthDate" 
                           Editable="true"
                           Mask="@(new DateMask("dd.MM.yyyy"))" 
                           DateFormat="dd.MM.yyyy"
                           Label=@L["BirthDate"] />
            <br />
            <MudTextField @bind-Value="@NewAuthor.ShortBio" 
                          Label=@L["ShortBio"]
                          Lines="5"
                          For=@(() => NewAuthor.ShortBio) />
        </MudForm>
    </DialogContent>
    <DialogActions>
        <MudButton Color="MudBlazor.Color.Secondary" 
                   OnClick="CloseCreateAuthorModal">
            @L["Cancel"]
        </MudButton>
        <MudButton Variant="Variant.Filled" 
                   Color="MudBlazor.Color.Primary" 
                   OnClick="CreateAuthorAsync">
            @L["Save"]
        </MudButton>
    </DialogActions>
</MudDialog>

<MudDialog @bind-IsVisible="_editAuthorDialogVisible">
    <TitleContent>
        <MudText Typo="Typo.h6">@EditingAuthor.Name</MudText>
    </TitleContent>
    <DialogContent>
        <MudForm Model="@EditingAuthor"
                 @ref="_editForm">
            <MudTextField @bind-Value="@EditingAuthor.Name" 
                          Label=@L["Name"]
                          For=@(() => EditingAuthor.Name) />
            <br />
            <MudDatePicker @bind-Date="@EditingAuthor.BirthDate" 
                           Editable="true"
                           Mask="@(new DateMask("dd.MM.yyyy"))" 
                           DateFormat="dd.MM.yyyy"
                           Label=@L["BirthDate"] />
            <br />
            <MudTextField @bind-Value="@EditingAuthor.ShortBio" 
                          Label=@L["ShortBio"]
                          Lines="5"
                          For=@(() => EditingAuthor.ShortBio) />
        </MudForm>
    </DialogContent>
    <DialogActions>
        <MudButton Color="MudBlazor.Color.Secondary" 
                   OnClick="CloseEditAuthorModal">
            @L["Cancel"]
        </MudButton>
        <MudButton Variant="Variant.Filled" 
                   Color="MudBlazor.Color.Primary" 
                   OnClick="UpdateAuthorAsync">
            @L["Save"]
        </MudButton>
    </DialogActions>
</MudDialog>
```

Then, update `Authors.razor.cs` code behing file with the following content:

```csharp
using System;
using System.Threading.Tasks;
using Acme.BookStore.Authors;
using Acme.BookStore.Permissions;
using Microsoft.AspNetCore.Authorization;
using MudBlazor;

namespace Acme.BookStore.Blazor.Pages
{
    public partial class Authors
    {
        private async Task<GridData<AuthorDto>> LoadServerData(GridState<AuthorDto> state)
        {
            var input = new GetAuthorListDto 
            { 
                Filter = "", 
                MaxResultCount = state.PageSize, 
                SkipCount = state.Page * state.PageSize
            };
            var result = await AuthorAppService.GetListAsync(input);
            return new()
            {
                Items = result.Items,
                TotalItems = (int)result.TotalCount
            };
        }

        private bool CanCreateAuthor { get; set; }
        private bool CanEditAuthor { get; set; }
        private bool CanDeleteAuthor { get; set; }

        private MudDataGrid<AuthorDto> _authorList;

        private CreateAuthorDto NewAuthor { get; set; }

        private bool _createAuthorDialogVisible;
        private bool _editAuthorDialogVisible;

        private MudForm _createForm;
        private MudForm _editForm;

        private Guid EditingAuthorId { get; set; }
        private UpdateAuthorDto EditingAuthor { get; set; }

        public Authors()
        {
            NewAuthor = new CreateAuthorDto();
            EditingAuthor = new UpdateAuthorDto();
        }

        protected override async Task OnInitializedAsync()
        {
            await SetPermissionsAsync();
        }

        private async Task SetPermissionsAsync()
        {
            CanCreateAuthor = await AuthorizationService
                .IsGrantedAsync(BookStorePermissions.Authors.Create);

            CanEditAuthor = await AuthorizationService
                .IsGrantedAsync(BookStorePermissions.Authors.Edit);

            CanDeleteAuthor = await AuthorizationService
                .IsGrantedAsync(BookStorePermissions.Authors.Delete);
        }

        private void OpenCreateAuthorModal()
        {
            NewAuthor = new CreateAuthorDto();
            _createAuthorDialogVisible = true;
        }

        private void CloseCreateAuthorModal()
        {
            _createAuthorDialogVisible = false;
        }

        private void OpenEditAuthorModal(AuthorDto author)
        {
            EditingAuthorId = author.Id;
            EditingAuthor = ObjectMapper.Map<AuthorDto, UpdateAuthorDto>(author);
            _editAuthorDialogVisible = true;
        }

        private async Task DeleteAuthorAsync(AuthorDto author)
        {
            var confirmMessage = L["AuthorDeletionConfirmationMessage", author.Name];
            if (!await Message.Confirm(confirmMessage))
            {
                return;
            }

            await AuthorAppService.DeleteAsync(author.Id);
            await _authorList.ReloadServerData();
        }

        private void CloseEditAuthorModal()
        {
            _editAuthorDialogVisible = false;
        }

        private async Task CreateAuthorAsync()
        {
            if (_createForm.IsValid)
            {
                await AuthorAppService.CreateAsync(NewAuthor);
                _createAuthorDialogVisible = false;
                await _authorList.ReloadServerData();
            }
        }

        private async Task UpdateAuthorAsync()
        {
            if (_editForm.IsValid)
            {
                await AuthorAppService.UpdateAsync(EditingAuthorId, EditingAuthor);
                _editAuthorDialogVisible = false;
                await _authorList.ReloadServerData();
            }
        }
    }
}
```

Change type of `BirthDate` property in `Author`, `AuthorDto`, `CreateAuthorDto`, `UpdateAuthorDto` from `DateTime` to `DateTime?`

Now, the __Authors__ page should look like this:

(screenshot)

## 14. Define `MudCrudPageBase` Class

Create `MudCrudPageBase.cs` file in `Volo.Abp.AspNetCore.Components.Web.BasicTheme/Components` and paste the following code:

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using JetBrains.Annotations;
using Localization.Resources.AbpUi;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.Extensions.Localization;
using MudBlazor;
using Volo.Abp.Application.Dtos;
using Volo.Abp.Application.Services;
using Volo.Abp.Authorization;
using Volo.Abp.BlazoriseUI.Components;
using Volo.Abp.AspNetCore.Components.Web.Extensibility.EntityActions;
using Volo.Abp.AspNetCore.Components.Web.Extensibility.TableColumns;

namespace Volo.Abp.AspNetCore.Components.Web.BasicTheme.Components;

public abstract class MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey>
    : MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey,
        PagedAndSortedResultRequestDto>
    where TAppService : ICrudAppService<
        TEntityDto,
        TKey>
    where TEntityDto : class, IEntityDto<TKey>, new()
{
}

public abstract class MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey,
        TGetListInput>
    : MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey,
        TGetListInput,
        TEntityDto>
    where TAppService : ICrudAppService<
        TEntityDto,
        TKey,
        TGetListInput>
    where TEntityDto : class, IEntityDto<TKey>, new()
    where TGetListInput : new()
{
}

public abstract class MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey,
        TGetListInput,
        TCreateInput>
    : MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TCreateInput>
    where TAppService : ICrudAppService<
        TEntityDto,
        TKey,
        TGetListInput,
        TCreateInput>
    where TEntityDto : IEntityDto<TKey>
    where TCreateInput : class, new()
    where TGetListInput : new()
{
}

public abstract class MudCrudPageBase<
        TAppService,
        TEntityDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput>
    : MudCrudPageBase<
        TAppService,
        TEntityDto,
        TEntityDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput>
    where TAppService : ICrudAppService<
        TEntityDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput>
    where TEntityDto : IEntityDto<TKey>
    where TCreateInput : class, new()
    where TUpdateInput : class, new()
    where TGetListInput : new()
{
}

public abstract class MudCrudPageBase<
        TAppService,
        TGetOutputDto,
        TGetListOutputDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput>
    : MudCrudPageBase<
        TAppService,
        TGetOutputDto,
        TGetListOutputDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput,
        TGetListOutputDto,
        TCreateInput,
        TUpdateInput>
    where TAppService : ICrudAppService<
        TGetOutputDto,
        TGetListOutputDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput>
    where TGetOutputDto : IEntityDto<TKey>
    where TGetListOutputDto : IEntityDto<TKey>
    where TCreateInput : class, new()
    where TUpdateInput : class, new()
    where TGetListInput : new()
{
}

public abstract class MudCrudPageBase<
        TAppService,
        TGetOutputDto,
        TGetListOutputDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput,
        TListViewModel,
        TCreateViewModel,
        TUpdateViewModel>
    : AbpComponentBase
    where TAppService : ICrudAppService<
        TGetOutputDto,
        TGetListOutputDto,
        TKey,
        TGetListInput,
        TCreateInput,
        TUpdateInput>
    where TGetOutputDto : IEntityDto<TKey>
    where TGetListOutputDto : IEntityDto<TKey>
    where TCreateInput : class
    where TUpdateInput : class
    where TGetListInput : new()
    where TListViewModel : IEntityDto<TKey>
    where TCreateViewModel : class, new()
    where TUpdateViewModel : class, new()
{
    [Inject] protected TAppService AppService { get; set; }
    [Inject] protected IStringLocalizer<AbpUiResource> UiLocalizer { get; set; }

    protected virtual int PageSize { get; } = LimitedResultRequestDto.DefaultMaxResultCount;

    protected TGetListInput GetListInput = new();
    protected TCreateViewModel NewEntity;
    protected TKey EditingEntityId;
    protected TUpdateViewModel EditingEntity;

    protected MudDataGrid<TListViewModel> DataGrid;

    protected bool CreateDialogVisible;
    protected bool EditDialogVisible;
    
    protected MudForm CreateForm;
    protected MudForm EditForm;

    protected DataGridEntityActionsColumn<TListViewModel> EntityActionsColumn;
    protected EntityActionDictionary EntityActions { get; set; }
    protected TableColumnDictionary TableColumns { get; set; }

    protected string CreatePolicyName { get; set; }
    protected string UpdatePolicyName { get; set; }
    protected string DeletePolicyName { get; set; }

    public bool HasCreatePermission { get; set; }
    public bool HasUpdatePermission { get; set; }
    public bool HasDeletePermission { get; set; }

    protected DialogOptions DialogOptions 
    {
        get => new()
        { 
            CloseButton = true,
            CloseOnEscapeKey = true,
            DisableBackdropClick = true
        };
    }

    protected MudCrudPageBase()
    {
        NewEntity = new TCreateViewModel();
        EditingEntity = new TUpdateViewModel();
        TableColumns = new TableColumnDictionary();
        EntityActions = new EntityActionDictionary();
    }

    protected async override Task OnInitializedAsync()
    {
        await SetPermissionsAsync();
        await SetEntityActionsAsync();
        await SetTableColumnsAsync();
        await InvokeAsync(StateHasChanged);
    }

    protected async override Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await base.OnAfterRenderAsync(firstRender);
            await SetToolbarItemsAsync();
            await SetBreadcrumbItemsAsync();
        }
    }

    protected virtual async Task SetPermissionsAsync()
    {
        if (CreatePolicyName != null)
        {
            HasCreatePermission = await AuthorizationService.IsGrantedAsync(CreatePolicyName);
        }

        if (UpdatePolicyName != null)
        {
            HasUpdatePermission = await AuthorizationService.IsGrantedAsync(UpdatePolicyName);
        }

        if (DeletePolicyName != null)
        {
            HasDeletePermission = await AuthorizationService.IsGrantedAsync(DeletePolicyName);
        }
    }

    private IReadOnlyList<TListViewModel> MapToListViewModel(IReadOnlyList<TGetListOutputDto> dtos)
    {
        if (typeof(TGetListOutputDto) == typeof(TListViewModel))
        {
            return dtos.As<IReadOnlyList<TListViewModel>>();
        }

        return ObjectMapper.Map<IReadOnlyList<TGetListOutputDto>, List<TListViewModel>>(dtos);
    }

    protected async Task<GridData<TListViewModel>> LoadServerData(GridState<TListViewModel> state)
    {
        if (GetListInput is PagedAndSortedResultRequestDto pageResultRequest)
        {
            pageResultRequest.MaxResultCount = state.PageSize;
            pageResultRequest.SkipCount = state.Page * state.PageSize;
        }
        var result = await AppService.GetListAsync(GetListInput);
        return new()
        {
            Items = MapToListViewModel(result.Items),
            TotalItems = (int)result.TotalCount
        };
    }

    protected virtual async Task OpenCreateModalAsync()
    {
        try
        {
            await CheckCreatePolicyAsync();

            NewEntity = new TCreateViewModel();

            // Mapper will not notify Blazor that binded values are changed
            // so we need to notify it manually by calling StateHasChanged
            await InvokeAsync(() =>
            {
                StateHasChanged();
                CreateDialogVisible = true;
            });
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
    }

    protected virtual Task CloseCreateModalAsync()
    {
        NewEntity = new TCreateViewModel();
        return InvokeAsync(() => { CreateDialogVisible = false; });
    }

    protected virtual async Task OpenEditModalAsync(TListViewModel entity)
    {
        try
        {
            await CheckUpdatePolicyAsync();

            var entityDto = await AppService.GetAsync(entity.Id);

            EditingEntityId = entity.Id;
            EditingEntity = MapToEditingEntity(entityDto);

            await InvokeAsync(() =>
            {
                StateHasChanged();
                EditDialogVisible = true;
            });
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
    }

    protected virtual TUpdateViewModel MapToEditingEntity(TGetOutputDto entityDto)
    {
        return ObjectMapper.Map<TGetOutputDto, TUpdateViewModel>(entityDto);
    }

    protected virtual TCreateInput MapToCreateInput(TCreateViewModel createViewModel)
    {
        if (typeof(TCreateInput) == typeof(TCreateViewModel))
        {
            return createViewModel.As<TCreateInput>();
        }

        return ObjectMapper.Map<TCreateViewModel, TCreateInput>(createViewModel);
    }

    protected virtual TUpdateInput MapToUpdateInput(TUpdateViewModel updateViewModel)
    {
        if (typeof(TUpdateInput) == typeof(TUpdateViewModel))
        {
            return updateViewModel.As<TUpdateInput>();
        }

        return ObjectMapper.Map<TUpdateViewModel, TUpdateInput>(updateViewModel);
    }

    protected virtual Task CloseEditModalAsync()
    {
        InvokeAsync(() => { EditDialogVisible = false; });
        return Task.CompletedTask;
    }

    protected virtual async Task CreateEntityAsync()
    {
        try
        {
            var validate = true;
            if (CreateForm is not null)
            {
                validate = CreateForm.IsValid;
            }
            if (validate)
            {
                await OnCreatingEntityAsync();

                await CheckCreatePolicyAsync();
                var createInput = MapToCreateInput(NewEntity);
                await AppService.CreateAsync(createInput);

                await OnCreatedEntityAsync();
            }
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
    }

    protected virtual Task OnCreatingEntityAsync()
    {
        return Task.CompletedTask;
    }

    protected virtual async Task OnCreatedEntityAsync()
    {
        NewEntity = new TCreateViewModel();
        await DataGrid.ReloadServerData();

        await InvokeAsync(() => { CreateDialogVisible = false; });
    }

    protected virtual async Task UpdateEntityAsync()
    {
        try
        {
            var validate = true;
            if (EditForm is not null)
            {
                validate = EditForm.IsValid;
            }
            if (validate)
            {
                await OnUpdatingEntityAsync();

                await CheckUpdatePolicyAsync();
                var updateInput = MapToUpdateInput(EditingEntity);
                await AppService.UpdateAsync(EditingEntityId, updateInput);

                await OnUpdatedEntityAsync();
            }
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
    }

    protected virtual Task OnUpdatingEntityAsync()
    {
        return Task.CompletedTask;
    }

    protected virtual async Task OnUpdatedEntityAsync()
    {
        await DataGrid.ReloadServerData();

        await InvokeAsync(() => { EditDialogVisible = false; });
    }

    protected virtual async Task DeleteEntityAsync(TListViewModel entity)
    {
        try
        {
            await CheckDeletePolicyAsync();
            await OnDeletingEntityAsync();
            await AppService.DeleteAsync(entity.Id);
            await OnDeletedEntityAsync();
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
    }

    protected virtual Task OnDeletingEntityAsync()
    {
        return Task.CompletedTask;
    }

    protected virtual async Task OnDeletedEntityAsync()
    {
        await DataGrid.ReloadServerData();
        await InvokeAsync(StateHasChanged);
        await Notify.Success(L["SuccessfullyDeleted"]);
    }

    protected virtual string GetDeleteConfirmationMessage(TListViewModel entity)
    {
        return UiLocalizer["ItemWillBeDeletedMessage"];
    }

    protected virtual async Task CheckCreatePolicyAsync()
    {
        await CheckPolicyAsync(CreatePolicyName);
    }

    protected virtual async Task CheckUpdatePolicyAsync()
    {
        await CheckPolicyAsync(UpdatePolicyName);
    }

    protected virtual async Task CheckDeletePolicyAsync()
    {
        await CheckPolicyAsync(DeletePolicyName);
    }

    /// <summary>
    /// Calls IAuthorizationService.CheckAsync for the given <paramref name="policyName"/>.
    /// Throws <see cref="AbpAuthorizationException"/> if given policy was not granted for the current user.
    ///
    /// Does nothing if <paramref name="policyName"/> is null or empty.
    /// </summary>
    /// <param name="policyName">A policy name to check</param>
    protected virtual async Task CheckPolicyAsync([CanBeNull] string policyName)
    {
        if (string.IsNullOrEmpty(policyName))
        {
            return;
        }

        await AuthorizationService.CheckAsync(policyName);
    }

    protected virtual ValueTask SetBreadcrumbItemsAsync()
    {
        return ValueTask.CompletedTask;
    }

    protected virtual ValueTask SetEntityActionsAsync()
    {
        return ValueTask.CompletedTask;
    }

    protected virtual ValueTask SetTableColumnsAsync()
    {
        return ValueTask.CompletedTask;
    }

    protected virtual ValueTask SetToolbarItemsAsync()
    {
        return ValueTask.CompletedTask;
    }
}
```

This class will be used to replace [AbpCrudPageBase](https://github.com/abpframework/abp/blob/dev/framework/src/Volo.Abp.BlazoriseUI/AbpCrudPageBase.cs).

## 15. Update __Books__ CRUD Page

Change type of `PublishDate` property in `Book`, `BookDto`, `CreateUpdateBookDto` from `DateTime` to `DateTime?`

Update `Books.razor` with the following content:

```razor
@page "/books"
@attribute [Authorize(BookStorePermissions.Books.Default)]
@using Acme.BookStore.Permissions
@using Microsoft.AspNetCore.Authorization
@using Volo.Abp.Application.Dtos
@using Acme.BookStore.Books
@using Acme.BookStore.Localization
@using Microsoft.Extensions.Localization
@using Volo.Abp.AspNetCore.Components.Web.BasicTheme.Components
@inherits MudCrudPageBase<IBookAppService, BookDto, Guid, PagedAndSortedResultRequestDto, CreateUpdateBookDto>
@inject AbpBlazorMessageLocalizerHelper<BookStoreResource> LH

<MudCard Elevation="8">
    <MudCardHeader>
        <MudGrid>
            <MudItem>
                <MudText Typo="Typo.h5">@L["Books"]</MudText>
            </MudItem>
            <MudSpacer />
            <MudItem>
                <MudButton Color="MudBlazor.Color.Primary"
                           Variant="Variant.Outlined"
                           Disabled="!HasCreatePermission"
                           OnClick="OpenCreateModalAsync">
                    @L["NewAuthor"]
                </MudButton>
            </MudItem>
        </MudGrid>
    </MudCardHeader>
    <MudCardContent>
        <MudDataGrid T="BookDto"
                     @ref="DataGrid"
                     Striped="true"
                     ServerData="LoadServerData">
            <Columns>
                <MudBlazor.Column T="BookDto"
                                  Field="@nameof(BookDto.Id)"
                                  Title="@L["Actions"]">
                    <CellTemplate>
                        @if (HasUpdatePermission)
                        {
                            <MudIconButton Icon="fas fa-edit" 
                                           OnClick="@(async (_) => { await OpenEditModalAsync(context); })"
                                           Size="MudBlazor.Size.Small" />
                        }
                        @if (HasDeletePermission)
                        {   
                            <MudIconButton Icon="fas fa-trash" 
                                           OnClick="@(async (e) => {await DeleteEntityAsync(context);})"
                                           Size="MudBlazor.Size.Small" />
                        }
                    </CellTemplate>
                </MudBlazor.Column>
                <MudBlazor.Column T="BookDto"
                                  Field="@nameof(BookDto.Name)"
                                  Title=@L["Name"] />
                <MudBlazor.Column T="BookDto"
                                  Field="@nameof(BookDto.Type)"
                                  Title=@L["Type"]>
                    <CellTemplate>
                        @L[$"Enum:BookType:{(int)context.Type}"]
                    </CellTemplate>
                </MudBlazor.Column>
                <MudBlazor.Column T="BookDto"
                                  Field="@nameof(BookDto.AuthorName)"
                                  Title=@L["AuthorName"] />
                <MudBlazor.Column T="BookDto"
                                  Field="@nameof(BookDto.PublishDate)"
                                  Title=@L["PublishDate"]>
                    <CellTemplate>
                        @if (@context.PublishDate.HasValue)
                            @context.PublishDate.Value.ToShortDateString()
                    </CellTemplate>
                </MudBlazor.Column>
                <MudBlazor.Column T="BookDto"
                                  Field="@nameof(BookDto.Price)"
                                  Title=@L["Price"] />
            </Columns>
        </MudDataGrid>
    </MudCardContent>
</MudCard>

<MudDialog @bind-IsVisible="CreateDialogVisible" 
           Options="DialogOptions">
    <TitleContent>
        <MudText Typo="Typo.h6">@L["NewBook"]</MudText>
    </TitleContent>
    <DialogContent>
        <MudForm Model="@NewEntity"
                 @ref="CreateForm">
            <MudSelect T="Guid" @bind-Value="NewEntity.AuthorId">
                <MudSelectItem Value="@Guid.Empty">
                    -
                </MudSelectItem>
                @foreach (var author in authorList)
                {
                    <MudSelectItem Value="@author.Id">
                        @author.Name
                    </MudSelectItem>
                }
            </MudSelect>
            <br />
            <MudTextField @bind-Value="@NewEntity.Name" 
                          Label=@L["Name"]
                          For=@(() => NewEntity.Name) />
            <br />
            <MudSelect T="BookType" @bind-Value="NewEntity.Type">
                @foreach (BookType bookTypeValue in Enum.GetValues(typeof(BookType)))
                {
                    <MudSelectItem Value="@bookTypeValue">
                        @L[$"Enum:BookType:{bookTypeValue}"]
                    </MudSelectItem>
                }
            </MudSelect>
            <br />
            <MudDatePicker @bind-Date="@NewEntity.PublishDate" 
                           Editable="true"
                           Mask="@(new DateMask("dd.MM.yyyy"))" 
                           DateFormat="dd.MM.yyyy"
                           Label=@L["PublishDate"] />
            <br />
            <MudNumericField @bind-Value="@NewEntity.Price" 
                             Label=@L["Price"]
                             Min=0f
                             HideSpinButtons="true"
                             For=@(() => NewEntity.Price) />
        </MudForm>
    </DialogContent>
    <DialogActions>
        <MudButton Color="MudBlazor.Color.Secondary" 
                   OnClick="CloseCreateModalAsync">
            @L["Cancel"]
        </MudButton>
        <MudButton Variant="Variant.Filled" 
                   Color="MudBlazor.Color.Primary" 
                   OnClick="CreateEntityAsync">
            @L["Save"]
        </MudButton>
    </DialogActions>
</MudDialog>

<MudDialog @bind-IsVisible="EditDialogVisible"
           Options="DialogOptions">
    <TitleContent>
        <MudText Typo="Typo.h6">@EditingEntity.Name</MudText>
    </TitleContent>
    <DialogContent>
        <MudForm Model="@EditingEntity"
                 @ref="EditForm">
            <MudSelect T="Guid" @bind-Value="EditingEntity.AuthorId">
                <MudSelectItem Value="@Guid.Empty">
                    -
                </MudSelectItem>
                @foreach (var author in authorList)
                {
                    <MudSelectItem Value="@author.Id">
                        @author.Name
                    </MudSelectItem>
                }
            </MudSelect>
            <br />
            <MudTextField @bind-Value="@EditingEntity.Name" 
                          Label=@L["Name"]
                          For=@(() => EditingEntity.Name) />
            <br />
            <MudSelect T="BookType" @bind-Value="EditingEntity.Type">
                @foreach (BookType bookTypeValue in Enum.GetValues(typeof(BookType)))
                {
                    <MudSelectItem Value="@bookTypeValue">
                        @L[$"Enum:BookType:{bookTypeValue}"]
                    </MudSelectItem>
                }
            </MudSelect>
            <br />
            <MudDatePicker @bind-Date="@EditingEntity.PublishDate" 
                           Editable="true"
                           Mask="@(new DateMask("dd.MM.yyyy"))" 
                           DateFormat="dd.MM.yyyy"
                           Label=@L["PublishDate"] />
            <br />
            <MudNumericField @bind-Value="@EditingEntity.Price" 
                             Label=@L["Price"]
                             Min=0f
                             HideSpinButtons="true"
                             For=@(() => EditingEntity.Price) />
        </MudForm>
    </DialogContent>
    <DialogActions>
        <MudButton Color="MudBlazor.Color.Secondary" 
                   OnClick="CloseEditModalAsync">
            @L["Cancel"]
        </MudButton>
        <MudButton Variant="Variant.Filled" 
                   Color="MudBlazor.Color.Primary" 
                   OnClick="UpdateEntityAsync">
            @L["Save"]
        </MudButton>
    </DialogActions>
</MudDialog>

@code
{
    //ADDED A NEW FIELD
    IReadOnlyList<AuthorLookupDto> authorList = Array.Empty<AuthorLookupDto>();

    public Books() // Constructor
    {
        CreatePolicyName = BookStorePermissions.Books.Create;
        UpdatePolicyName = BookStorePermissions.Books.Edit;
        DeletePolicyName = BookStorePermissions.Books.Delete;
    }

    //GET AUTHORS ON INITIALIZATION
    protected override async Task OnInitializedAsync()
    {
        await base.OnInitializedAsync();
        authorList = (await AppService.GetAuthorLookupAsync()).Items;
    }
}
```

## Next

Update built-in Blazor pages to use MudBlazor components.

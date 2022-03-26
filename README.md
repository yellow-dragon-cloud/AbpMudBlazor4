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




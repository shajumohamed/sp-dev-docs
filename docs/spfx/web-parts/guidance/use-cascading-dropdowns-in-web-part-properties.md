---
title: Use cascading dropdowns in web part properties
description: Create cascading dropdown controls in the SharePoint client-side web part property pane without developing a custom property pane control.
ms.date: 02/11/2022
ms.localizationpriority: high
---
# Use cascading dropdowns in web part properties

When designing the property pane for your SharePoint client-side web parts, you may have one web part property that displays its options based on the  value selected in another property. This scenario typically occurs when implementing cascading dropdown controls. In this article, you learn how to create cascading dropdown controls in the web part property pane without developing a custom property pane control.

![Item dropdown disabled and web part placeholder communicating loading updated list of item options](../../../images/react-cascading-dropdowns-loading-indicator-when-loading-items.png)

The source of the working web part is available on GitHub at [sp-dev-fx-webparts/samples/react-custompropertypanecontrols/](https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-custompropertypanecontrols).

> [!NOTE]
> Before following the steps in this article, be sure to [set up your SharePoint client-side web part development environment](../../set-up-your-development-environment.md).

## Create new project

1. Start by creating a new folder for your project:

    ```console
    md react-cascadingdropdowns
    ```

1. Go to the project folder:

    ```console
    cd react-cascadingdropdowns
    ```

1. In the project folder, run the SharePoint Framework Yeoman generator to scaffold a new SharePoint Framework project:

    ```console
    yo @microsoft/sharepoint
    ```

1. When prompted, enter the following values (*select the default option for all prompts omitted below*):

    - **What is your solution name?**: react-cascadingdropdowns
    - **Which type of client-side component to create?**: WebPart
    - **What is your Web part name?**: List items
    - **Which template would you like to use?**: React

1. Open your project folder in your code editor. This article uses Visual Studio Code in the steps and screenshots, but you can use any editor that you prefer.

## Define a web part property to store the selected list

You'll build a web part that displays list items from a selected SharePoint list. Users can select a list in the web part property pane. To store the selected list, create a new web part property named `listName`.

1. In the code editor, open the **src/webparts/listItems/ListItemsWebPartManifest.json** file. Replace the default `description` property with a new property named `listName`.

    ```json
    {
      ...
      "preconfiguredEntries": [{
        ...
        "properties": {
          "listName": ""
        }
      }]
    }
    ```

1. Open the **src/webparts/listItems/ListItemsWebPart.ts** file, and replace the `IListItemsWebPartProps` interface with the following:

    ```typescript
    export interface IListItemsWebPartProps {
      listName: string;
    }
    ```

1. In the **src/webparts/listItems/ListItemsWebPart.ts** file, change the `render()` method to:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      public render(): void {
        const element: React.ReactElement<IListItemsProps> = React.createElement(ListItems, {
          listName: this.properties.listName
        });

        ReactDom.render(element, this.domElement);
      }
      // ...
    }
    ```

1. Update `getPropertyPaneConfiguration()` to:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...

      protected getPropertyPaneConfiguration(): IPropertyPaneConfiguration {
        return {
          pages: [
            {
              header: {
                description: strings.PropertyPaneDescription
              },
              groups: [
                {
                  groupName: strings.BasicGroupName,
                  groupFields: [
                    PropertyPaneTextField('listName', {
                      label: strings.ListNameFieldLabel
                    })
                  ]
                }
              ]
            }
          ]
        };
      }

      // ...
    }
    ```

1. In the **src/webparts/listItems/loc/mystrings.d.ts** file, change the `IListItemsStrings` interface to:

    ```typescript
    declare interface IListItemsStrings {
      PropertyPaneDescription: string;
      BasicGroupName: string;
      ListNameFieldLabel: string;
    }
    ```

1. In the **src/webparts/listItems/loc/en-us.js** file, add the missing definition for the `ListNameFieldLabel` string:

    ```javascript
    define([], function() {
      return {
        "PropertyPaneDescription": "Description",
        "BasicGroupName": "Group Name",
        "ListNameFieldLabel": "List"
      }
    });
    ```

1. In the **src/webparts/listItems/components/ListItems.tsx** file, change the contents of the `render()` method to:

    ```tsx
    public render(): JSX.Element {
      const {
        listName
      } = this.props;

      return (
        <section className={`${styles.listItems} ${hasTeamsContext ? styles.teams : ''}`}>
          <div className={styles.welcome}>
            <img alt="" src={isDarkTheme ? require('../assets/welcome-dark.png') : require('../assets/welcome-light.png')} className={styles.welcomeImage} />
            <h2>Well done, {escape(userDisplayName)}!</h2>
            <div>{environmentMessage}</div>
            <div>List name: <strong>{escape(listName)}</strong></div>
          </div>
        </section>
      );
    }
    ```

1. In the **src/webparts/listItems/components/IListItemsProps.ts** file, change the `IListItemsProps` interface to:

    ```typescript
    export interface IListItemsProps {
      listName: string;
    }
    ```

1. Run the following command to verify that the project is running:

    ```console
    gulp serve
    ```

1. In the web browser, add the **List items** web part to the canvas and open its properties. Verify that the value set for the **List** property is displayed in the web part body.

    ![Web part showing the value of the listName property](../../../images/react-cascading-dropdowns-web-part-first-run.png)

## Populate the dropdown with SharePoint lists to choose from

At this point, a user specifies which list the web part should use by manually entering the list name. This is error-prone, and ideally you want users to choose one of the lists existing in the current SharePoint site.

### Use dropdown control to render the listName property

1. In the `ListItemsWebPart` class, add a reference to the `PropertyPaneDropdown` class in the top section of the web part. Replace the import clause that loads the `PropertyPaneTextField` class with:

    ```typescript
    import {
      IPropertyPaneConfiguration,
      PropertyPaneTextField,
      PropertyPaneDropdown,
      IPropertyPaneDropdownOption
    } from '@microsoft/sp-property-pane';
    ```

1. In the `ListItemsWebPart` class, add a new variable named `lists` to store information about all available lists in the current site:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      private lists: IPropertyPaneDropdownOption[];
      // ...
    }
    ```

1. Add a new class variable named `listsDropdownDisabled`. This variable determines whether the list dropdown is enabled or not. Until the web part retrieves the information about the lists available in the current site, the dropdown should be disabled.

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      private listsDropdownDisabled: boolean = true;
      // ...
    }
    ```

1. Change the `getPropertyPaneConfiguration()` method to use the dropdown control to render the `listName` property:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...

      protected getPropertyPaneConfiguration(): IPropertyPaneConfiguration {
        return {
          pages: [
            {
              header: {
                description: strings.PropertyPaneDescription
              },
              groups: [
                {
                  groupName: strings.BasicGroupName,
                  groupFields: [
                    PropertyPaneDropdown('listName', {
                      label: strings.ListNameFieldLabel,
                      options: this.lists,
                      disabled: this.listsDropdownDisabled
                    })
                  ]
                }
              ]
            }
          ]
        };
      }

    }
    ```

1. Run the following command to verify that it's working as expected:

    ```console
    gulp serve
    ```

    ![The listName property rendered in the web part property pane using a dropdown control](../../../images/react-cascading-dropdowns-listname-property-pane-dropdown.png)

### Show available lists in the list dropdown

Previously, you associated the dropdown control of the `listName` property with the `lists` class property. Because you haven't loaded any values into it yet, the **List** dropdown in the web part property pane remains disabled. In this section, you'll extend the web part to load the information about available lists.

1. In the `ListItemsWebPart` class, add a method to load available lists. You'll use mock data, but you could also call the SharePoint REST API to retrieve the list of available lists from the current web. To simulate loading options from an external service, the method uses a two-second delay.

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...

      private loadLists(): Promise<IPropertyPaneDropdownOption[]> {
        return new Promise<IPropertyPaneDropdownOption[]>((resolve: (options: IPropertyPaneDropdownOption[]) => void, reject: (error: any) => void) => {
          setTimeout((): void => {
            resolve([{
              key: 'sharedDocuments',
              text: 'Shared Documents'
            },
            {
              key: 'myDocuments',
              text: 'My Documents'
            }]);
          }, 2000);
        });
      }
    }
    ```

1. Load information about available lists into the list dropdown. In the `ListItemsWebPart` class, override the `onPropertyPaneConfigurationStart()` method by using the following code:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...

      protected onPropertyPaneConfigurationStart(): void {
        this.listsDropdownDisabled = !this.lists;

        if (this.lists) {
          return;
        }

        this.context.statusRenderer.displayLoadingIndicator(this.domElement, 'lists');

        this.loadLists()
          .then((listOptions: IPropertyPaneDropdownOption[]): void => {
            this.lists = listOptions;
            this.listsDropdownDisabled = false;
            this.context.propertyPane.refresh();
            this.context.statusRenderer.clearLoadingIndicator(this.domElement);
            this.render();
          });
      }
      // ...
    }
    ```

    The `onPropertyPaneConfigurationStart()` method is called by the SharePoint Framework after the web part property pane for the web part has been opened.

    - First, the method checks if the information about the lists available in the current site has been loaded.
    - If the list information is loaded, the list dropdown is enabled.
    - If the list information about lists hasn't been loaded yet, the loading indicator is displayed, which informs the user that the web part is loading information about lists.

    ![Loading indicator displayed in web part while loading information about available lists](../../../images/react-cascading-dropdowns-loading-indicator-when-loading-list-info.png)

    After the information about available lists has been loaded, the method assigns the retrieved data to the `lists` class variable, from which it can be used by the list dropdown.

    Next, the dropdown is enabled allowing the user to select a list. By calling `this.context.propertyPane.refresh()`, the web part property pane is refreshed and it reflects the latest changes to the list dropdown.

    After list information is loaded, the loading indicator is removed by a call to the `clearLoadingIndicator()` method. Because calling this method clears the web part user interface, the `render()` method is called to force the web part to re-render.

1. Run the following command to confirm that everything is working as expected:

    ```console
    gulp serve
    ```

    When you add a web part to the canvas and open its property pane, you should see the lists dropdown filled with available lists for the user to choose from.

    ![List dropdown in the web part property pane showing the available lists](../../../images/react-cascading-dropdowns-list-dropdown-available-lists.png)

## Allow users to select an item from the selected list

When building web parts, you often need to allow users to choose an option from a set of values determined by a previously selected value, such as choosing a country/region based on the selected continent, or choosing a list item from a selected list. This user experience is often referred to as cascading dropdowns. Using the standard SharePoint Framework client-side web parts capabilities, you can build cascading dropdowns in the web part property pane. To do this, you extend the previously built web part with the ability to choose a list item based on the previously selected list.

![List item dropdown open in the web part property pane](../../../images/react-cascading-dropdowns-item-dropdown-list-items.png)

### Add item web part property

1. In the code editor, open the **src/webparts/listItems/ListItemsWebPart.manifest.json** file. To the `properties` section, add a new property named `itemName` so that it appears as follows:

    ```json
    {
      // ...
      "properties": {
        "listName": "",
        "itemName": ""
      }
      // ...
    }
    ```

1. Change the code in the **src/webparts/listItems/IListItemsWebPartProps.ts** file to:

    ```typescript
    export interface IListItemsWebPartProps {
      listName: string;
      itemName: string;
    }
    ```

1. Change the code in the **src/webparts/listItems/components/IListItemsProps.ts** file to:

    ```typescript
    export interface IListItemsProps {
      listName: string;
      itemName: string;
    }
    ```

1. In the **src/webparts/listItems/ListItemsWebPart.ts** file, change the code of the `render()` method to:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      public render(): void {
        const element: React.ReactElement<IListItemsProps> = React.createElement(ListItems, {
          listName: this.properties.listName,
          itemName: this.properties.itemName
        });

        ReactDom.render(element, this.domElement);
      }
      // ...
    }
    ```

1. In the **src/webparts/listItems/loc/mystrings.d.ts** file, change the `IListItemsStrings` interface to:

    ```typescript
    declare interface IListItemsStrings {
      PropertyPaneDescription: string;
      BasicGroupName: string;
      ListNameFieldLabel: string;
      ItemNameFieldLabel: string;
    }
    ```

1. In the **src/webparts/listItems/loc/en-us.js** file, add the missing definition for the `ItemNameFieldLabel` string.

    ```javascript
    define([], function() {
      return {
        "PropertyPaneDescription": "Description",
        "BasicGroupName": "Group Name",
        "ListNameFieldLabel": "List",
        "ItemNameFieldLabel": "Item"
      }
    });
    ```

### Render the value of the item web part property

In the **src/webparts/listItems/components/ListItems.tsx** file, change the `render()` method to:

```tsx
export default class ListItems extends React.Component<IListItemsProps, {}> {
  public render(): JSX.Element {
    const {
      listName,
      itemName
    } = this.props;

    return (
      <section className={`${styles.listItems} ${hasTeamsContext ? styles.teams : ''}`}>
        <div className={styles.welcome}>
          <img alt="" src={isDarkTheme ? require('../assets/welcome-dark.png') : require('../assets/welcome-light.png')} className={styles.welcomeImage} />
          <h2>Well done, {escape(userDisplayName)}!</h2>
          <div>{environmentMessage}</div>
          <div>List name: <strong>{escape(listName)}</strong></div>
          <div>Item name: <strong>{escape(itemName)}</strong></div>
        </div>
      </section>
    );
  }
}
```

### Allow users to choose the item from a list

Similar to how users can select a list by using a dropdown, they can select the item from the list of available items.

1. In the `ListItemsWebPart` class, add a new variable named `items`, which you use to store information about all available items in the currently selected list.

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      private items: IPropertyPaneDropdownOption[];
      // ...
    }
    ```

1. Add a new class variable named `itemsDropdownDisabled`. This variable determines whether the items dropdown should be enabled or not. Users can select an item only after they selected a list.

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      private itemsDropdownDisabled: boolean = true;
      // ...
    }
    ```

1. Change the `getPropertyPaneConfiguration()` method to use the dropdown control to render the `itemName` property.

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      protected getPropertyPaneConfiguration(): IPropertyPaneConfiguration {
        return {
          pages: [
            {
              header: {
                description: strings.PropertyPaneDescription
              },
              groups: [
                {
                  groupName: strings.BasicGroupName,
                  groupFields: [
                    PropertyPaneDropdown('listName', {
                      label: strings.ListNameFieldLabel,
                      options: this.lists,
                      disabled: this.listsDropdownDisabled
                    }),
                    PropertyPaneDropdown('itemName', {
                      label: strings.ItemNameFieldLabel,
                      options: this.items,
                      disabled: this.itemsDropdownDisabled
                    })
                  ]
                }
              ]
            }
          ]
        };
      }
    }
    ```

1. Run the following command to verify that it's working as expected:

    ```console
    gulp serve
    ```

    ![The itemName property rendered in the web part property pane using a dropdown control](../../../images/react-cascading-dropdowns-itemname-property-pane-dropdown.png)

### Show items available in the selected list in the item dropdown

Previously, you defined a dropdown control to render the `itemName` property in the web part property pane. Next, you extend the web part to load the information about items available in the selected list, and show the items in the item dropdown.

1. Add method to load list items. In the **src/webparts/listItems/ListItemsWebPart.ts** file, in the `ListItemsWebPart` class, add a new method to load available list items from the selected list. (Like the method for loading available lists, you use mock data.)

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      private loadItems(): Promise<IPropertyPaneDropdownOption[]> {
        if (!this.properties.listName) {
          // resolve to empty options since no list has been selected
          return Promise.resolve();
        }

        const wp: ListItemsWebPart = this;

        return new Promise<IPropertyPaneDropdownOption[]>((resolve: (options: IPropertyPaneDropdownOption[]) => void, reject: (error: any) => void) => {
          setTimeout(() => {
            const items = {
              sharedDocuments: [
                {
                  key: 'spfx_presentation.pptx',
                  text: 'SPFx for the masses'
                },
                {
                  key: 'hello-world.spapp',
                  text: 'hello-world.spapp'
                }
              ],
              myDocuments: [
                {
                  key: 'isaiah_cv.docx',
                  text: 'Isaiah CV'
                },
                {
                  key: 'isaiah_expenses.xlsx',
                  text: 'Isaiah Expenses'
                }
              ]
            };
            resolve(items[wp.properties.listName]);
          }, 2000);
        });
      }
    }
    ```

    The `loadItems()` method returns mock list items for the previously selected list. When no list has been selected, the method resolves the promise without any data.

1. Load information about available items into the item dropdown. In the `ListItemsWebPart` class, extend the `onPropertyPaneConfigurationStart()` method to load items for the selected list:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      protected onPropertyPaneConfigurationStart(): void {
        this.listsDropdownDisabled = !this.lists;
        this.itemsDropdownDisabled = !this.properties.listName || !this.items;

        if (this.lists) {
          return;
        }

        this.context.statusRenderer.displayLoadingIndicator(this.domElement, 'options');

        this.loadLists()
          .then((listOptions: IPropertyPaneDropdownOption[]): Promise<IPropertyPaneDropdownOption[]> => {
            this.lists = listOptions;
            this.listsDropdownDisabled = false;
            this.context.propertyPane.refresh();
            return this.loadItems();
          })
          .then((itemOptions: IPropertyPaneDropdownOption[]): void => {
            this.items = itemOptions;
            this.itemsDropdownDisabled = !this.properties.listName;
            this.context.propertyPane.refresh();
            this.context.statusRenderer.clearLoadingIndicator(this.domElement);
            this.render();
          });
      }
      // ...
    }
    ```

    When initializing, the web part first determines if the items dropdown should be enabled or not. If the user previously selected a list, they can select an item from that list. If no list was selected, the item dropdown is disabled.

    You extended the previously defined code, which loads the information about available lists, to load the information about items available in the selected list. The code then assigns the retrieved information to the `items` class variable for use by the item dropdown. Finally, the code clears the loading indicator and allows the user to start working with the web part.

1. Run the following command to confirm that everything is working as expected:

    ```console
    gulp serve
    ```

    As required, initially the item dropdown is disabled, requiring users to select a list first. But at this point, even after a list has been selected, the item dropdown remains disabled.

    ![Item dropdown disabled even after a list has been selected](../../../images/react-cascading-dropdowns-list-selected-item-disabled.png)

1. Update web part property pane after selecting a list. When a user selects a list in the property pane, the web part should update, enabling the item dropdown and showing the list of items available in the selected list.

    In the **ListItemsWebPart.ts** file, in the `ListItemsWebPart` class, override the `onPropertyPaneFieldChanged()` method with the following code:

    ```typescript
    export default class ListItemsWebPart extends BaseClientSideWebPart<IListItemsWebPartProps> {
      // ...
      protected onPropertyPaneFieldChanged(propertyPath: string, oldValue: any, newValue: any): void {
        if (propertyPath === 'listName' &&
            newValue) {
          // push new list value
          super.onPropertyPaneFieldChanged(propertyPath, oldValue, newValue);
          // get previously selected item
          const previousItem: string = this.properties.itemName;
          // reset selected item
          this.properties.itemName = undefined;
          // push new item value
          this.onPropertyPaneFieldChanged('itemName', previousItem, this.properties.itemName);
          // disable item selector until new items are loaded
          this.itemsDropdownDisabled = true;
          // refresh the item selector control by repainting the property pane
          this.context.propertyPane.refresh();
          // communicate loading items
          this.context.statusRenderer.displayLoadingIndicator(this.domElement, 'items');

          this.loadItems()
            .then((itemOptions: IPropertyPaneDropdownOption[]): void => {
              // store items
              this.items = itemOptions;
              // enable item selector
              this.itemsDropdownDisabled = false;
              // clear status indicator
              this.context.statusRenderer.clearLoadingIndicator(this.domElement);
              // re-render the web part as clearing the loading indicator removes the web part body
              this.render();
              // refresh the item selector control by repainting the property pane
              this.context.propertyPane.refresh();
            });
        }
        else {
          super.onPropertyPaneFieldChanged(propertyPath, oldValue, newValue);
        }
      }
      // ...
    }
    ```

    After the user selected a list, the web part persists the newly selected value. Because the selected list changed, the web part resets the previously selected list item. Now that a list is selected, the web part property pane loads list items for that particular list. While loading items, the user can't select an item.

    After the items for the selected list are loaded, they're assigned to the **items** class variable from where they can be referenced by the item dropdown. Now that the information about available list items is available, the item dropdown is enabled allowing users to choose an item. The loading indicator is removed, which clears the web part body that is why the web part should re-render. Finally, the web part property pane refreshes to reflect the latest changes.

    ![Item dropdown in the web part property pane showing available list items for the selected list](../../../images/react-cascading-dropdowns-item-dropdown-list-items.png)

## See also

- [Build custom controls for the property pane](./build-custom-property-pane-controls.md)

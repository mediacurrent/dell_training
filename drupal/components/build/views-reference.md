# Views reference

Let's add two new fields to the Homepage content type. These will be views reference fields in order to display the blocks we created in the Blog posts view.

Be sure to enable the **Views reference field** module before proceeding.

1. From Drupal's admin toolbar, select **Structure &gt; Content types &gt; Homepage &gt; Manage fields**
2. Click the **Add field**  button
3. From the **Add new field** dropdown, under the _Reference_ group, select **Views reference**
4. Type **Blog articles list** as the label and click the **Save and continue** button
5. Limit number of values is **Unlimited**
6. Click the **Save field settings** button
7. Under **View display plugins to allow** select **Block**
8. Under **Preselect View Options** select **Blog posts**
9. **Save** your changes
10. In Manage display hide the field label.

## **Adding the views block to the homepage**

1. Edit the existing homepage node
2. In the **Blog articles** field, type **Blog posts** and select **Blog posts** when displayed
3. For the **Display ID** select **Featured content**
4. Click **Add another item**
5. This time select **From our blog** as the **Display ID**
6. **Save** your node



[Source](http://atendesigngroup.com/blog/removing-duplicate-content-across-multiple-drupal-views "Permalink to Removing Duplicate Content Across Multiple Drupal Views")

# Removing Duplicate Content Across Multiple Drupal Views

[Views][1] is an indispensable and powerful module at the heart of Drupal that you can use to quickly generate structured tables or lists of consistently formatted content, and filter and group that content by simple or complex logic. But in pushing Views to do ever more complex and useful things, we can sort of paint ourselves into a corner sometimes. For instance, I have many times created multiple Views displays on a single page that contain overlapping content. My homepage has a Views display of manually curated content, using [Nodequeue][2] or a similar module. On the same homepage, I have a Views display of news content that shows the most recent content. Since the two different Views displays pull from the same bucket of content, it is very possible to have duplicate content across the displays. Here is an example:

![Before image showing overlapping content.][3]

_Notice the underlined duplicate titles across the two Views displays._

**This is what we want:**

![After image showing the deduped Views display.][4]

_Notice the missing featured titles from the deduped Views display._

By creating a custom Drupal module and utilizing a [Views hook][5], we can remove the duplicate content across the two Views displays. We programmatically check exactly which pieces of content are in one View, and we feed that information to a filter in the second View that excludes it.

### Before diving into my example, I want to cover a few assumptions I'm making about you.

* You are using Drupal 7
* You are familiar with Views module
* You know how to install modules
* You know at least a touch of PHP

## Steps to Follow Along

[View Example Code on Github][6]

### Step 1

My example code assumes that you have created two Views displays.

* **Featured** \- A View display of manually curated content. This display will be used to generate a list of content to exclude from our automated Views display.
* **Automated** \- A View display of news content that shows the most recent content. This display will accept a list of content to be excluded.

You can of course adapt the Views displays to your exact needs.

After creating the Views you wish to use, you'll need to know the machine name of the View and View display.

One way to retrieve these names is from the view edit URL. While editing your view, notice the URL:

`/admin/structure/views/view/automated_news/edit/block`

In my case, automated_news is the view name and block is the view display name.

_Make a note of your machine names for Step 3_

### Step 2

On the view you wish to dedup or exclude content from, you'll need to add and configure a contextual filter.

1. Navigate to edit the automated content view
2. Under "Advanced" &amp; "Contextual Filters", click add and select "Content: Nid (The node ID.)"
3. Select "Provide default value" and choose "Fixed value".
4. Leave the Fixed value empty as we'll provide this in code
5. Under "More" select "Allow multiple values" and "Exclude"
6. Save the view

### Step 3

Enable your custom module that contains the deduping code. You are welcome to [download the example module on Github][6] and use it, or add the code to an existing custom module if it makes more sense. In any case, you'll need to customize the module a little bit to work with your Views.

1. Update the machine name variables from Step 1. Find the `$dedup_views` array and alter the values for `featured_view_name`, `featured_view_display`, `automated_view_name` and `automated_view_display`
2. Save your module
3. Enable your module
4. Clear your Drupal cache

If everything was configured correctly, you should see your Views displays properly deduped.

## Code Explained

[View Example Code on Github][6]

The code relies on [`hook_views_pre_view()`][7], a Views hook. Using this hook, we can pass values to the Views display contextual filter set in Step 2. Here is a version where content IDs (NIDs) 1, 2, 5 &amp; 6 are manually being passed to a view for exclusion.

    /**
      * @implements hook_views_pre_view().
      *
      * https://api.drupal.org/api/views/views.api.php/function/hook_views_pre_view/7
      */
    function hook_views_pre_view(&$view, &$display_id, &$args){
      // Check for the specific View name and display
      if ($view->name == 'automated_news' && $display_id == 'block') {
        $args[] = 1+2+5+6;
      }
    }

There are many ways you could dynamically build a list of NIDs you wish to exclude. In my example, we are loading another Views display to build a list of NIDs. The function [`views_get_view()`][8] loads a Views display in code and provides access to the result set.

    // Load the view
    // https://api.drupal.org/api/views/views.module/function/views_get_view/7
    $view = views_get_view('automated_news');
    $view->set_display('block');
    $view->pre_execute();
    $view->execute();
    
    // Get the results
    $results = $view->result;

Drupal Views is a powerful module and I like the ability to extend it even further using the extensive Views hooks API. In the case of my example, we can keep using Views with writing complex database queries.

[1]: https://www.drupal.org/project/views
[2]: https://www.drupal.org/project/nodequeue
[3]: https://www.evernote.com/shard/s201/sh/6c37faaa-4384-4a4e-85c0-0f3fb04aa4eb/fe4405a678537c5e30dba0876ddb8698/deep/0/views_before.png
[4]: https://www.evernote.com/shard/s201/sh/78978421-b83e-4450-b026-1458c15ee6e9/2e82a5d85d56efbe7e0342328e242f00/deep/0/views_after.png
[5]: https://api.drupal.org/api/views/views.api.php/group/views_hooks/7
[6]: https://github.com/joelsteidl/dedup_views
[7]: https://api.drupal.org/api/views/views.api.php/function/hook_views_pre_view/7
[8]: https://api.drupal.org/api/views/views.module/function/views_get_view/7
  
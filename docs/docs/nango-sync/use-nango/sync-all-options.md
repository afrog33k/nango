import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Create Syncs

Adding a Sync to Nango is quick & easy. If you have not read the [core concepts](/nango-sync/use-nango/core-concepts.md) yet please do so first: From here on out we assume you are familiar with Syncs and how they work together with your application.

## 3 quick steps to add a new Sync

We recommend the following steps when you add a new Sync to Nango:
1. Make sure the API request works as expected (using Postman, a script, CLI etc.)
2. Look at the Sync options below and configure them for your Sync
3. Run your code once and make sure Nango syncs your data as expected

## Sync options {#sync-options}

This example shows you all the possible configuration options for a Nango Sync. 

**All configuration fields are optional** (though you may need to provide the relevant ones for the external API request to succeed). 

If you want to see some examples of them in action take a look at the [real world examples](/nango-sync/real-world-examples.md) page.

For all the `path` parameters you can use `.` characters to reference keys in nested objects: `paging.next.after`

<Tabs groupId="programming-language">
  <TabItem value="node" label="Node SDK">

```ts
import {Nango, NangoHttpMethod} from '@nangohq/node-client'

let config = {
    //==================
    // Config for the HTTP request to the 3rd party API
    //==================
    method: NangoHttpMethod.Get,    // The HTTP method of the external REST API endpoint (GET, POST, etc.). Default: GET.
    headers: {                      // HTTP headers to send along with every request to the external API (e.g. auth header).
        'Accept': 'application/json'
    },
    body: {                         // HTTP body to send along with every request to the external API.
        'mykey': 'A great value'
    },
    query_params: {
        'mykey': 'A great value'    // URL query params to send along with every request to the external API.
    },

    //==================
    // Fetching records & uniquely identifying them
    //==================
    response_path: 'data.results',  // The path to the result objects inside the external API response.
    unique_key: 'profile.email',    // The key in the result objects used for deduping (e.g. email, id) + enables Upsert syncing mode.
    metadata: {                     // Will be attached to every synced record. 1 column per key.
        user_id: 123,
        company: 'supercorp'
    }

    //==================
    // Deletion modes
    //==================
    soft_delete: false,             // Defaults to false. Soft Delete Mode will keep deleted records in the db and add the deletion date in a 'deleted_at' column.

    //==================
    // Pagination
    //==================

    // Pagination strategy 1: Next page cursor in response metadata
    paging_cursor_request_path: 'after',   // Provide the cursor request path for fetching the next page.
    paging_cursor_metadata_response_path: 'paging.next.after',   // Use a field in the response as cursor for the next page.

    // Pagination strategy 2: Next page cursor in response's last object
    paging_cursor_request_path: 'after',   // Provide the cursor request path for fetching the next page.
    paging_cursor_object_response_path: 'id', // Use a field of the response's last object as cursor for the next page.

    // Pagination strategy 3: Next page URL in response body
    paging_url_path: 'next',        // Use a field in the response as URL for the next page.

    // Pagination strategy 4: Next page URL in response header
    paging_header_link_rel: 'next', // Use the Link Header to fetch the next page.

    //==================
    // JSON-to-SQL schema mapping (for details see "Schema mappings" in the sidebar)
    //==================
    auto_mapping: true,             // Automatically map JSON objects returned from external APIs to SQL columns. Default: true.
    mapped_table: 'example_table'   // The name of the table in the database where the results should be stored.

    //==================
    // Sync frequency
    //==================
    cron: "* * * * *",              // Run Sync jobs on a cron schedule.
    frequency: "1 hour",            // Alternatively, run Sync jobs with an unaligned frequency, using natural language.

    //==================
    // Authentication (only needed for OAuth, leverages Nango)
    //==================
    nango_connection_id: "user1",  // The ID of the connection registered with Nango
    nango_provider_config_key: "hubspot",  // The key of the provider configuration registered with Nango
    
    //==================
    // Convenience features
    //==================
    max_total: 100,                 // Limit the total number of total objects synced for testing purposes.
    friendly_name: 'My Sync',       // Human readable name, will be used in logs & observability.
};

// Add the Sync
new Nango().sync('https://api.example.com/my/endpoint?query=A+query', config);
```
  </TabItem>
  <TabItem value="curl" label="REST API (curl)">

  ```json
  curl --request POST \
--url http://localhost:3003/v1/syncs \
 --header "Content-type: application/json" \
 --data '{
    "url": "https://api.example.com/my/endpoint?query=A+query",
    "method": "GET",
    "headers": { "Accept": "application/json"},
    "body": { "mykey": "A great value"},
    "query_params": { "mykey": "A great value"},
    "response_path": "data.results",
    "unique_key": "profile.email",
    "paging_cursor_request_path": "after",
    "paging_cursor_metadata_response_path": "next",
    "paging_url_path": "next",
    "paging_cursor_object_response_path": "id",
    "paging_header_link_rel": "next",
    "auto_mapping": true,
    "frequency": 15,
    "nango_connection_id": "user1",
    "nango_provider_config_key": "hubspot",
    "max_total": 100,
    "friendly_name": "My Sync",
    "metadata": { "company_id": 123 }
  }'

#==================
# Config for the HTTP request to the 3rd party API
#==================
#
# - url: The URL of the external API endpoint
# - method: The HTTP method of the external REST API endpoint (GET, POST, etc.). Default: GET.
# - headers: HTTP headers to send along with every request to the external API (e.g. auth header).
# - body: HTTP body to send along with every request to the external API.
# - query_params: URL query params to send along with every request to the external API.
#
#==================
# Fetching records & uniquely identifying them
#==================
#
# - response_path: The path to the result objects inside the external API response.
# - unique_key: The key in the result objects used for deduping (e.g. email, id) + enables Full Refresh + Upsert syncing mode.
# - metadata: Will be attached to every synced record. 1 column per key.
#
#==================
# Pagination
#==================
#
# Pagination strategy 1: Next page cursor in response metadata
# - paging_cursor_request_path: Provide the cursor request path for fetching the next page.
# - paging_cursor_metadata_response_path: Use a field in the response as cursor for the next page.
# 
# Pagination strategy 2: Next page cursor in response's last object
# - paging_cursor_request_path: Provide the cursor request path for fetching the next page.
# - paging_cursor_object_response_path: Use a field of the response's last object as cursor for the next page.
#
# Pagination strategy 3: Next page URL in response body
# - paging_url_path: Use a field in the response as URL for the next page.
#
# Pagination strategy 4: Next page URL in response header
# - paging_header_link_rel: Use the Link Header to fetch the next page.
#
#==================
# JSON-to-SQL schema mapping (for details see "Schema mappings" in the sidebar)
#==================
#
# - auto_mapping: Automatically map JSON objects returned from external APIs to SQL columns. Default: true.
# - mapped_table: The name of the table in the database where the results should be stored.
#
#==================
# Sync frequency
#==================
#
# - frequency: Sync interval in minutes. Default: 60
#
#==================
# Authentication (only needed for OAuth, leverages Nango)
#==================
#
# - nango_connection_id: The ID of the connection registered with Nango
# - nango_provider_config_key: The key of the provider configuration registered with Nango
#
#==================
# Convenience features
#==================
#
# - max_total: Limit the total number of total objects synced for testing purposes.
# - friendly_name: Human readable name, will be used in logs & observability.
#

  ```
  </TabItem>
</Tabs>

## Problems with your Sync? We are here to help!

If you need help or run into issues, please reach out! We are online and responsive all day on the [Slack Community](https://nango.dev/slack).
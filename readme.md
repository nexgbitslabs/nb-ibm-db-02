#!/bin/bash

# Output CSV file
OUTPUT="eventhub_details.csv"

# Header
echo "SubscriptionId,ResourceGroup,NamespaceName,EventHubName,PartitionCount,RetentionDays" > "$OUTPUT"

# Get current subscription ID
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# Get all Event Hub namespaces in the subscription
NAMESPACES=$(az eventhubs namespace list --query "[].{name:name, resourceGroup:resourceGroup}" -o json)

# Loop through each namespace
echo "$NAMESPACES" | jq -c '.[]' | while read -r ns; do
    NAMESPACE_NAME=$(echo "$ns" | jq -r '.name')
    RESOURCE_GROUP=$(echo "$ns" | jq -r '.resourceGroup')

    # Get all Event Hubs in the namespace
    EVENT_HUBS=$(az eventhubs eventhub list --resource-group "$RESOURCE_GROUP" --namespace-name "$NAMESPACE_NAME" -o json)

    echo "$EVENT_HUBS" | jq -c '.[]' | while read -r hub; do
        EVENT_HUB_NAME=$(echo "$hub" | jq -r '.name')
        PARTITION_COUNT=$(echo "$hub" | jq -r '.partitionCount')
        RETENTION_DAYS=$(echo "$hub" | jq -r '.messageRetentionInDays')

        echo "$SUBSCRIPTION_ID,$RESOURCE_GROUP,$NAMESPACE_NAME,$EVENT_HUB_NAME,$PARTITION_COUNT,$RETENTION_DAYS" >> "$OUTPUT"
    done
done

echo "✅ CSV written to $OUTPUT"

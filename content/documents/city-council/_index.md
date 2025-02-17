---
title: "City Council Documents"
params:
  lastSyncDate: "2025-02-16"
---

{{< city-council-sync >}}

This page contains City Council and Committee meeting PDF documents from 2022 to 2025 for the City of Windsor. You can search, sort, and filter the data. The documents are synced once a week automatically using the [civic-code doc-search](https://github.com/dntiontk/civic-code?tab=readme-ov-file#doc-search) tool.

<!-- Latest DataTables CSS -->
<link href="https://cdn.datatables.net/2.2.2/css/dataTables.min.css" rel="stylesheet">
<link href="https://cdn.datatables.net/buttons/3.0.1/css/buttons.dataTables.min.css" rel="stylesheet">

<!-- Required JS -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.datatables.net/2.2.2/js/dataTables.min.js"></script>
<script src="https://cdn.datatables.net/buttons/3.0.1/js/dataTables.buttons.min.js"></script>
<script src="https://cdn.datatables.net/buttons/3.0.1/js/buttons.html5.min.js"></script>

<style>
.filters {
    margin: 20px 0;
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}
.filters label {
    margin-right: 5px;
    font-weight: bold;
}
.filters select {
    padding: 5px;
    border-radius: 4px;
    min-width: 200px;
    background-color: var(--background);
    color: var(--color);
    border: 1px solid var(--border-color);
}
.dataTables_wrapper {
    font-size: 0.9em;
}
table.dataTable {
    background-color: var(--background);
    color: var(--color);
    border-color: var(--border-color);
}
table.dataTable td {
    padding: 8px 10px;
    border-color: var(--border-color);
}
table.dataTable thead th {
    background-color: var(--background);
    color: var(--color);
    border-color: var(--border-color);
    font-weight: bold;
}
table.dataTable tbody tr {
    background-color: var(--background);
}
table.dataTable tbody tr:hover {
    background-color: var(--hover-color);
}
.document-type {
    font-size: 0.9em;
    opacity: 0.8;
}
.dataTables_wrapper .dataTables_length,
.dataTables_wrapper .dataTables_filter,
.dataTables_wrapper .dataTables_info,
.dataTables_wrapper .dataTables_processing,
.dataTables_wrapper .dataTables_paginate {
    color: var(--color) !important;
}
.dataTables_wrapper .dataTables_filter input {
    background-color: var(--background);
    color: var(--color);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    padding: 5px;
}
.dataTables_wrapper .dataTables_length select {
    background-color: var(--background);
    color: var(--color);
    border: 1px solid var(--border-color);
}
.dataTables_wrapper .dataTables_paginate .paginate_button {
    background-color: var(--background) !important;
    color: var(--color) !important;
    border: 1px solid var(--border-color) !important;
}
.dataTables_wrapper .dataTables_paginate .paginate_button:hover {
    background-color: var(--hover-color) !important;
    color: var(--color) !important;
}
.dataTables_wrapper .dataTables_paginate .paginate_button.current {
    background-color: var(--hover-color) !important;
    color: var(--color) !important;
}
.download-link {
    color: var(--color);
    text-decoration: underline;
}
.download-link:hover {
    opacity: 0.8;
}
</style>

<div class="filters">
    <div>
        <label for="meetingTypeFilter">Committee:</label>
        <select id="meetingTypeFilter">
            <option value="">All Committees</option>
        </select>
    </div>
    <div>
        <label for="yearFilter">Year:</label>
        <select id="yearFilter">
            <option value="">All Years</option>
        </select>
    </div>
    <div>
        <label for="documentTypeFilter">Document Type:</label>
        <select id="documentTypeFilter">
            <option value="">All Types</option>
        </select>
    </div>
</div>

<table id="documentsTable" class="display">
    <thead>
        <tr>
            <th>Committee</th>
            <th>Date</th>
            <th>Document</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody id="tableBody">
    </tbody>
</table>

<script>
const table = new DataTable('#documentsTable', {
    pageLength: 25,
    order: [[1, 'desc']],
    dom: 'lBfrtip',
    buttons: ['csv', 'excel'],
    language: {
        search: "Search all columns:"
    },
    ajax: {
        url: '/documents/city-council/documents.json',
        dataSrc: 'items'
    },
    columns: [
        { 
            data: 'meeting.name' 
        },
        { 
            data: 'date',
            render: (data, type, row) => {
                if (type === 'display') {
                    const date = new Date(data);
                    return date.toLocaleDateString('en-GB', {
                        weekday: 'long',
                        year: 'numeric',
                        month: 'long',
                        day: 'numeric'
                    });
                }
                return data;
            }
        },
        { 
            data: 'name',
            render: (data, type, row) => {
                const docType = data.split('.')[0].toLowerCase().includes('agenda') ? 'Agenda' :
                               data.split('.')[0].toLowerCase().includes('minutes') ? 'Minutes' :
                               data.split('.')[0].toLowerCase().includes('consolidated') ? 'Consolidated Agenda' :
                               'Other';
                return `<strong>${docType}</strong><div class="document-type">${data}</div>`;
            }
        },
        { 
            data: 'link',
            render: (data, type, row) => `<a href="${data}" target="_blank" class="download-link">Download</a>`
        }
    ],
    initComplete: function(settings, json) {
        // Populate filters
        const meetingTypes = new Set(json.items.map(item => item.meeting.name));
        const years = new Set(json.items.map(item => new Date(item.date).getFullYear()));
        const documentTypes = new Set(json.items.map(item => {
            const name = item.name.toLowerCase();
            return name.includes('agenda') ? 'Agenda' :
                   name.includes('minutes') ? 'Minutes' :
                   name.includes('consolidated') ? 'Consolidated Agenda' :
                   'Other';
        }));
        
        populateFilter('meetingTypeFilter', meetingTypes);
        populateFilter('yearFilter', Array.from(years).sort((a, b) => b - a));
        populateFilter('documentTypeFilter', documentTypes);
        
        // Add filter functionality
        $('.filters select').on('change', function() {
            table.draw();
        });
        
        // Custom filtering function
        $.fn.dataTable.ext.search.push(function(settings, data, dataIndex) {
            const meetingType = $('#meetingTypeFilter').val();
            const year = $('#yearFilter').val();
            const docType = $('#documentTypeFilter').val();
            
            const rowMeetingType = data[0];
            const rowDate = new Date(data[1]);
            const rowDocType = data[2];
            
            const meetingMatch = !meetingType || rowMeetingType === meetingType;
            const yearMatch = !year || rowDate.getFullYear().toString() === year;
            const docMatch = !docType || rowDocType.includes(docType);
            
            return meetingMatch && yearMatch && docMatch;
        });
    }
});

function populateFilter(id, values) {
    const filter = document.getElementById(id);
    Array.from(values).sort().forEach(value => {
        const option = document.createElement('option');
        option.value = value;
        option.textContent = value;
        filter.appendChild(option);
    });
}
</script>

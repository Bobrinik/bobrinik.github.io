---
layout: post
title: "How to download portfolio composition from Wealthsimple"
date: 2024-04-03
tags: [tutorial, wealthsimple, hacky_solution]
---

In short, people are asking for capabilities to export data from Wealthsimple so that they can track it in Excel or do some Python modelling. So far, the solutions are to either use Wealthica that is using some unknown API or some sort of a crawler to extract that information (you would need to give it your creds, not ideal) you would also need to pay for ability to download it from them or you can manually copy paste the information.

## Solution

> Grease Monkey is a popular browser extension that allows users to customize the functionality and appearance of websites they visit. It works with various web browsers, including Google Chrome, Mozilla Firefox, and others. Grease Monkey uses user scripts, which are small JavaScript programs, to modify the behavior of web pages. Grease Monkey works by injecting user scripts into web pages as they are loaded in your browser.  - ChatGPT

The idea is to inject script into webpage that would add functionality which is lacking. That script would get necessary data from the loaded webpage and put it into a CSV. It would also add a download button to the webpage so that person could download it.

That's how it looks.

```jsx
// ==UserScript==
// @name          jQuery Example
// @require       https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.1/jquery.min.js
// ==/UserScript==

function getFormattedDate() {
    var dateObj = new Date();
    var year = dateObj.getFullYear();
    var month = ("0" + (dateObj.getMonth() + 1)).slice(-2); // getMonth() is zero-based
    var day = ("0" + dateObj.getDate()).slice(-2);

    return `${year}-${month}-${day}`;
}

window.onload = function() {
    setTimeout(function () {
      jQuery(document).ready(function($) {
          let downloadButton = document.createElement("button");
          downloadButton.innerHTML = "Download CSV";
          downloadButton.id = "csvButton";
          downloadButton.style.padding = "20px"; 
        
          document.body.insertBefore(downloadButton, document.body.firstChild);

          function generateCSV() {
              let separator = ",";
              let csvContent = [];
              let header = ['Security', 'Name', 'Total_Value', 'Quantity', 'All_Time_Return', 'Per_All_time_Return', 'Today_Price', 'Per_Today_Price'];
              
              csvContent.push(header.join(separator));
                          
              $("tbody tr").each(function () {
                  let row = [];
                  $(this).find("td").each(function () {
                      $(this).find("p").each(function() {
                          row.push($(this).text());
                      });
                  });
                
                  if(row.length == 9) {
                    row = row.slice(1);
                  }
                  console.log(row);
                  csvContent.push(row.join(separator));
              });
              return csvContent.join("\n");
          }

          document.getElementById("csvButton").addEventListener("click", function () {
              let accountName = $(".knseRw > div:nth-child(1)").text();
              let csvContent = generateCSV();
              var hiddenElement = document.createElement('a');
              hiddenElement.href = 'data:text/csv;charset=utf-8,' + encodeURI(csvContent);
              hiddenElement.target = '_blank';
              hiddenElement.download = accountName+'_portfolio_'+getFormattedDate()+'.csv';
              hiddenElement.click();
          });
      });
    }, 5000);
}
```

You can read more and follow instructions [here](https://github.com/Bobrinik/wealthsimple_utilities/tree/main).


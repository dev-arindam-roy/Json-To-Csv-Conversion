# JSON TO CSV DOWNLOAD

## An application on json to csv conversion & download

### Check The Application

[https://dev-arindam-roy.github.io/Json-To-Csv-Conversion/](https://dev-arindam-roy.github.io/Json-To-Csv-Conversion/)

```js
$(document).ready(function () {

    window.addEventListener('load', () => {
        registerSW();
    });

    // Register the Service Worker
    async function registerSW() {
        if ('serviceWorker' in navigator) {
            try {
                await navigator.serviceWorker.register('assets/pwa/serviceworker.js');
            }
            catch (e) {
                console.log('SW registration failed');
            }
        }
    }

    /** Copy the raw json */
    $('#copyJsonBtn').on('click', async function() {
        let copyJson = document.getElementById('convertedJson');
        await window.navigator.clipboard.writeText(copyJson.value);
        copyJson.select();
        displayToast();
    });

    /** SweetAlert2 loading */
    const displayLoading = (timer = 3000, title = 'Please Wait...', text = "System Processing Your Request") => {
        Swal.fire({
            title: title,
            text: text,
            allowEscapeKey: false,
            allowOutsideClick: false,
            timer: timer,
            didOpen: () => {
                Swal.showLoading()
            }
        });
    }

    /** SweetAlert2 like toast */
    const displayToast = () => {
        Swal.fire({
            position: 'top-end',
            icon: 'success',
            title: 'Its copied!',
            showConfirmButton: false,
            timer: 1000
        });
    }

    $("#frmx").validate({
        errorClass: 'onex-error',
        errorElement: 'div',
        rules: {
            json_file: {
                required: true,
                extension: 'json'
            }
        },
        messages: {
            json_file: {
                required: 'Please upload json file',
                extension: 'Only accept .json',
                accept: 'Only accept .json'
            }
        },
        errorPlacement: function (error, element) {
            if(element.hasClass('onex-select2')) {
                error.insertAfter(element.parent().find('span.select2-container'));
            } else {
                error.insertAfter(element);
            }
        },
        highlight: function (element) {
            $(element).removeClass('is-valid').addClass('is-invalid');
            $(element).parent().find('.onex-form-label').addClass('onex-error-label');
            if(element.type == 'select-one') {
                $(element).next('span.select2-container').addClass('select2-custom-error');
            }
        },
        unhighlight: function (element) {
            $(element).removeClass('is-invalid').addClass('is-valid');
            $(element).parent().find('.onex-form-label').removeClass('onex-error-label');
        },
        submitHandler: function (form) {
            uploadJsonFile();
            return false;
        }
    });

    function actionButtons(jsonObj) {
        if(jsonObj.length) {
            $('#downloadJsonBtn').removeClass('disabled');
            $('#copyJsonBtn').removeClass('disabled');
            $('#downloadCsvBtn').removeClass('disabled');
        } else {
            $('#downloadJsonBtn').addClass('disabled');
            $('#copyJsonBtn').addClass('disabled');
            $('#downloadCsvBtn').addClass('disabled');
        }
    }

    /*Json Content To CSV content*/
    function jsonToCsv(jsonData) {
        console.log(jsonData);
        let csv = '';
        // Get the headers
        let headers = Object.keys(jsonData[0]);
        csv += headers.join(',') + '\n';
        // Add the data
        jsonData.forEach(function (row) {
            let data = headers.map(header => JSON.stringify(row[header])).join(',');
            csv += data + '\n';
        });
        return csv;
    }

    let jsonFile = document.querySelector('#jsonFileUpload');
    
    /*Get Json File Content From Uploader*/
    async function uploadJsonFile() {
        displayLoading();
        let file = jsonFile.files[0];
        //console.log(file);
        let reader = new FileReader(file);
        reader.readAsText(file);
        reader.onload = async(e) => {
            let resultData = e.target.result;
            let jsonContent = await JSON.parse(resultData);
            console.log(jsonContent);
            actionButtons(jsonContent);
            $('#convertedJson').val(JSON.stringify(jsonContent, null, 4));
            $('#json-renderer').jsonViewer(jsonContent);
            $('.second-phase').show();
        }
    }

    /*Download as JSON*/
    $('#downloadJsonBtn').on('click', function() {
        const anchor = document.createElement('a');
        anchor.href = 'data:application/json;charset=utf-8,' + encodeURIComponent($('#convertedJson').val());
        anchor.download = new Date().toJSON() + '.json' ;
        document.body.appendChild(anchor);
        anchor.click();
        document.body.removeChild(anchor);
    });

    /*Download as CSV*/
    $('#downloadCsvBtn').on('click', function() {
        let content = jsonToCsv(JSON.parse($('#convertedJson').val()));
        let blob = new Blob([content], { type: 'text/csv' });
        let url = window.URL.createObjectURL(blob);
        let a = document.createElement('a');
        a.href = url;
        a.download = new Date().toJSON() + '.csv' ;
        document.body.appendChild(a);
        a.click();
    });
});
```
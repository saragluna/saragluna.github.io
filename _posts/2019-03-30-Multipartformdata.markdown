---
layout:     post
title:      "multipart/form-data"
subtitle:   "How Tomcat Parse Multipart Request?"
date:       "2019-03-30 18:00:00"
author:     "Sarag"
header-img: "img/post-bg-2019.jpg"
tags:
    - Java
---



In this post I want to talk about what is multipart/form-data and how web servers like Tomcat process Multipart request.

## Multipart/form-data Definition

### Boundary

The parts are delimited with a boundary delimiter, constructed using CRLF, "—", and the value of "boundary" parameter. The boundary is supplied as a "boundary" parameter to the multipart/form-data type. The boundary delimiter MUST NOT appear inside any of the encapsulated parts, and it is often necessary to enclose the "boundary" parameter values in quotes in the Content-Type header field.



### Content-Disposition Header Field for Each Part

Each Part MUST contain a `Content-Disposition` field where the disposition type is "form-data". The Content-Disposition header field MUST also contain an additional parameter of "name"; the value of the "name" parameter is the original field name from the form (possibly encoded). 

> Content-Disposition: form-data; name="key"

For form data that represents the content of a file, a name for the file SHOULD be supplied as well, by using a "filename" parameter of the Content-Disposition header field. The file **isn't mandatory** for cases where the file name isn't available or is meaningless or private; this might result, for example, when selection or drag-or-drop is used when the form data content is dirctly from a device.



### Multiple Files for One Form Field

The form data for a form field might include multiple files.

**To match widely deployed implementations**, multiple files **MUST be sent by supplying each file in a seperate part** but all with the same "name" parameter.



### Content-Type Header Field for Each Part

Each part MAY have an (optional) "Content-Type" header field, which defaults to "text/plain". If the contents of a file are to be sent, the file data SHOULD be labeled with an appropriate media type, if known, or "application/octet-stream".

> Content-Disposition: form-data; name="file"; filename="test-compatible.pdf"
> **Content-Type**: application/octet-stream



### The Charset Parameter for "text/plain" Form Data

In the case where the form data is text, the charset parameter for the "text/plain" Content-Type MAY be used to indicate the character encoding used in that part. For example, a form with a text field in which a user typed "Joe owes \<eu\>100", where \<eu> is the Euro symbol, might have form data returned as:

> --AaB03x
> content-disposition: form-data; name="field1"
> content-type: text/plain;charset=UTF-8
> content-transfer-encoding: quoted-printable
> Joe owes =E2=82=AC100.
> —AaB03x

In practice, many widely deployed implementations do supply a charset parameter in each part, but rather, they rely on the notion of a "default charset" for a multipart/form-data instance.

### Example

```js
Request Headers
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarybNxUaiz2BYOw4K8u

Form Data
------WebKitFormBoundarybNxUaiz2BYOw4K8u
Content-Disposition: form-data; name="name"

test-compatible.pdf
------WebKitFormBoundarybNxUaiz2BYOw4K8u
Content-Disposition: form-data; name="key"

abcdefgkey
------WebKitFormBoundarybNxUaiz2BYOw4K8u
Content-Disposition: form-data; name="success_action_status"

200
------WebKitFormBoundarybNxUaiz2BYOw4K8u
Content-Disposition: form-data; name="file"; filename="test-compatible.pdf"
Content-Type: application/octet-stream


------WebKitFormBoundarybNxUaiz2BYOw4K8u--
```



## How Tomcat Parse multipart/form

### Source Code

#### RequestsContext

> Abstracts access to the request information needed for file uploads. This interface should be implemented for each type of request that may be handled by FileUpload, such as servlets and portlets.

![image-20190330180412548](/img/in-post/multipartform/requests_context.png)



#### FileUpload

> High level API for processing file uploads.
>
> This class handles multiple files per single HTML widget, sent using multipart/mixed encoding type, as specified by RFC 1867 . Use parseRequest(RequestContext) to acquire a list of FileItems associated with a given HTML widget.
> How the data for individual parts is stored is determined by the factory used to create them; a given part may be in memory, on disk, or somewhere else.              

​                           ![image-20190330174215674](/img/in-post/multipartform/FileUploadBase.png)   



#### FileUpload and FileItermFacotry

![image-20190330174431401](/img/in-post/multipartform/fileupload_2.png)

```java
// FileUpload parse request
public List<FileItem> parseRequest(RequestContext ctx)
            throws FileUploadException {
        List<FileItem> items = new ArrayList<>();
        boolean successful = false;
        try {
            FileItemIterator iter = getItemIterator(ctx);
            FileItemFactory fac = getFileItemFactory();
            if (fac == null) {
                throw new NullPointerException("No FileItemFactory has been set.");
            }
            while (iter.hasNext()) {
                final FileItemStream item = iter.next();
                // Don't use getName() here to prevent an InvalidFileNameException.
                final String fileName = ((FileItemIteratorImpl.FileItemStreamImpl) item).name;
                FileItem fileItem = fac.createItem(item.getFieldName(), item.getContentType(), item.isFormField(), fileName);
                items.add(fileItem);
                try {
                    Streams.copy(item.openStream(), fileItem.getOutputStream(), true);
                } catch (FileUploadIOException e) {
                    throw (FileUploadException) e.getCause();
                } catch (IOException e) {
                    throw new IOFileUploadException(String.format("Processing of %s request failed. %s", MULTIPART_FORM_DATA, e.getMessage()), e);
                }
                final FileItemHeaders fih = item.getHeaders();
                fileItem.setHeaders(fih);
            }
            successful = true;
            return items;
        } catch (FileUploadIOException e) {
            throw (FileUploadException) e.getCause();
        } catch (IOException e) {
            throw new FileUploadException(e.getMessage(), e);
        } finally {
            if (!successful) {
                for (FileItem fileItem : items) {
                    try {
                        fileItem.delete();
                    } catch (Exception ignored) {
                        // ignored TODO perhaps add to tracker delete failure list somehow?
                    }
                }
            }
        }
    }
```



#### How to use in Servlet?

```java
//The following is a brief example of typical usage in a servlet, storing the uploaded files on disk.

public void doPost(HttpServletRequest req, HttpServletResponse res) {
   DiskFileItemFactory factory = new DiskFileItemFactory();
   // maximum size that will be stored in memory
   factory.setSizeThreshold(4096);
   // the location for saving data that is larger than getSizeThreshold()
   factory.setRepository(new File("/tmp"));

   ServletFileUpload upload = new ServletFileUpload(factory);
   // maximum size before a FileUploadException will be thrown
   upload.setSizeMax(1000000);

   List fileItems = upload.parseRequest(req);
   // assume we know there are two files. The first file is a small
   // text file, the second is unknown and is written to a file on
   // the server
   Iterator i = fileItems.iterator();
   String comment = ((FileItem)i.next()).getString();
   FileItem fi = (FileItem)i.next();
   // filename on the client
   String fileName = fi.getName();
   // save comment and filename to database
   ...
   // write the file
   fi.write(new File("/www/uploads/", fileName));
 }
```



## Why Does Tomcat Save the Files to Disk? 

Someone asked me a question that "Why does the web server have to store the file to disk (or memory,  depending on the file size) when it can process the stream directly?

My answer to this question is: 

1. To implement a common use web server, the server itself won't know what does the user want to do with the stream, the stream could be blocked by some computation work, which is not IO efficiency.
2. When the user want to process IO stream more efficiently, he can implement another function for it.
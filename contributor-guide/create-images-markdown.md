<properties
    pageTitle="Create images in markdown"
    description="Explains how to create images in markdown according to guidelines set for the Azure repositories."
    services=""
    solutions=""
    documentationCenter=""
    authors="kenhoff"
    manager="ilanas"
    editor="tysonn"/>

<tags
    ms.service="contributor-guide"
    ms.devlang=""
    ms.topic="article"
    ms.tgt_pltfrm=""
    ms.workload=""
    ms.date="06/25/2015"
    ms.author="kenhoff" />

# Create images in markdown

## Image folder creation and link syntax

To use images in your ACOM article, you need to create a folder in the following location:

    ./articles/service-directory/media/article-name/` directory.

For example:

    ./articles/app-service/media/app-service-enterprise-multichannel-apps/`.

After you create the folder and added images to it, use the following syntax to create images in your article:

```
![Alt image text](./media/article-name/your-image-filename.png)
```
Example:

See [the markdown template](https://raw.githubusercontent.com/Azure/azure-content-pr/master/markdown%20templates/markdown-template-for-new-articles.md) for an example.  The image call references in this markdown template are designed so the calls are made to image references at the bottom of the template.

## Guidelines specific to azure.microsoft.com

Screenshots are currently encouraged if it's not possible to include repro steps. Do write your content so that the content can stand without the screenshots if necessary.

Use the following guidelines when creating and including art files:
- Do not share art files across documents. Copy the file you need and add it to the media folder for your specific topic. Sharing between files is discouraged because  it is easier to remove deprecated content and images which keeps the repo clean.

- .png files are highly preferred over other formats.

- Use red squares of the default width provided in Paint (5 px) to call attention to particular elements in screenshots.  

    Example:
    
    ![This is an example of a red square used as a callout.](./media/create-images-markdown/gs13noauth.png)

- Avoid whitespace on edges of screenshots.  If you crop a screenshot in a way that leaves white background at the edges, add a single pixel gray border around the image.  If using Paint, use the lighter gray in the default color pallete (0xC3C3C3). If using some other graphic app, the RGB color is R195, G195, 195. You can easily add a gray border around an image in Visio--to do this, select the image, select Line, and ensure the the correct color is set, and then change the line weight to 1 1/2 pt.  Screenshots should have a 1-pixel-wide gray border so that white areas of the screenshot do not blur into the web page.

    Example:

    ![This is an example of a gray border around whitespace.](./media/create-images-markdown/agent.png)

- Try not to make an image too wide.  Images will be automatically resized if they are too wide. However, the resizing sometimes causes fuzziness, so we recommend that you limit the width of your images to 780 px, and manually resize images before submission if necessary.

- Show command outputs in screenshots.  If your article includes steps where the user is working within a shell, it's useful to show command output in screenshots. In this case, restricting your shell width to about 72 characters generally ensures that your image will remain within the 780 px width guideline. Before taking a screenshot of output, resize the window so that just the relevant command and output is shown (optionally with a blank line on either side).

- Take whole screenshots of windows when possible. When taking a screenshot of a browser window, resize your browser window to 780 px wide or less, and keep the height of the browser window as short as possible such that your application fits within the window.

    Example:

    ![This is an example of a browser window screenshot.](./media/create-images-markdown/helloworldlocal.png)

- Use caution with what information is revealed in screenshots.  Do not reveal internal company information or personal information.

- In conceptual art or diagrams, use the official icons in the Cloud and Enterprise symbol and icon set. A public set is available at http://aka.ms/CnESymbols.

###Contributors' Guide Links

- [Overview article](./../README.md)
- [Index of guidance articles](./contributor-guide-index.md)

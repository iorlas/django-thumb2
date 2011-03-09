_This is a my fork of original module [django-thumbs](http://code.google.com/p/django-thumbs/), which was created by Antonio Melé(see http://django.es) and maintained till Jun 08 2009(latest commit date). Great thanks to Antotio!_

**The easiest way to create thumbnails for your images with Django. Works with any StorageBackend.**

## Features

1. Easy to integrate in your code (no database changes, works as an ImageField)
2. Works perfectly with any StorageBackend
3. Generates thumbnails after image is uploaded into memory
4. Deletes thumbnails when the image file is deleted
5. Provides easy access to the thumbnails' URLs (similar method as with ImageField)

django-thumb2 requires django >1.1. Perfectly works on 1.2.

## Installation

1. Download `thumbs.py`
2. Import it in your models.py and replace `ImageField` with `ImageWithThumbsField` in your model
3. Add a sizes attribute with a list of sizes you want to use for the thumbnails
4. Make sure your have defined `MEDIA_URL` in your `settings.py`
5. That's it!

## Working example

_models.py_

    from django.db import models
    from thumbs import ImageWithThumbsField
    class Person(models.Model):
        photo = ImageWithThumbsField(upload_to='images', sizes=((125,125),(200,200)))
        second_photo = ImageWithThumbsField(upload_to='images')

In this example we have a `Person` model with 2 image fields.

You can see the field `second_photo` doesn't have a `sizes` attribute. This field works exactly the same way as a normal `ImageField`.

The field `photo` has a `sizes` attribute specifying desired sizes for the thumbnails. This field works the same way as `ImageField` but it also creates the desired thumbnails when uploading a new file and deletes the thumbnails when deleting the file.

With `ImageField` you retrieve the URL for the image with: `someone.photo.url` With `ImageWithThumbsField` you retrieve it the same way. You also retrieve the URL for every thumbnail specifying its size: In this example we use `someone.photo.url_125x125` and `someone.photo.url_200x200` to get the URL of both thumbnails.

## Uninstall

At any time you can go back and use `ImageField` again without altering the database or anything else. Just replace `ImageWithThumbsField` with `ImageField` again and make sure you delete the sizes attribute. Everything will work the same way it worked before using `django-thumbs`. Just remember to delete generated thumbnails in the case you don't want to have them anymore.

## Inner logic: resizing.

### Quad thumbs(if you requested NxN size of thumbnail)
Unknown, lol. I need to review this module more time, but not now, sorry :C

### Unquad thumbs(if you requested NxJ size)
_There is difference beyond original module, but i think it is more sensible._
It resizes image by minimal side. If you need 100x50 thumb, but image was 300x500, images resizes into 200x50.
Next step - cropping. It crops image to needed thumbnail size, making it centered.

Original logic keeps in the PIL, because django-thumb just calls `PIL.thumbnail` with needed size as argument.
So, it just fits image into needed size, what makes whitespace in the bottom or right size of image. No! It doesn't makes whitespaces, it just crops all canvas, not a layer! So, if you'll try to use PIL.thumbnail as it is by 300x500 image and 200x50 thumb size, you'll get an 30x50 image.

Dont believe me? Just check it:

    >>> img = Image.open("check.png")
    >>> img
    <PIL.PngImagePlugin.PngImageFile image mode=RGB size=300x500 at 0x7F906D696B90>
    >>> img.size
    (300, 500)
    >>> img.thumbnail((200, 50), Image.ANTIALIAS)
    >>> img
    <PIL.PngImagePlugin.PngImageFile image mode=RGB size=30x50 at 0x7F906D696B90>
    >>> img.size
    (30, 50)

This is no sense, itn't? Thats why i've reworked this myself.



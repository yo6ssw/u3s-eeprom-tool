---
layout: post
title: Circle/Box Interesection Revised
excerpt: "This article continues our endeavors in the circle/aabb collision detection world presenting an extremely fast (if not the fastest) method to detect them."
---

After working out the method presented in the previous article, a discussion with \_3yE\_ on _#coders_ opened my eyes towards an extremely fast method who suffers no accuracy loss.

![grid](/public/images/grid.png "The zones grid around the box")

According to the above picture, the center point of the circle can belong to any of the 9 zones. Code follows for deciding the zone of the circle:

{% highlight c++ %}
int xZone = circle.x < (box.x - box.halfWidth) ? 0 : 
           (circle.x > (box.x + box.halfWidth) ? 2 : 1);

int yZone = circle.y < (box.y - box.halfHeight) ? 0 : 
           (circle.y > (box.y + box.halfHeight) ? 2 : 1);

int zone = xZone + 3*yZone;
{% endhighlight %}

We notice there are three cases we need to handle:

1. circle center is inside the box (zone 4)
2. circle center is in a corner zone (zones 0, 2, 6, 8)
3. circle center is in a side zone (zones 1, 3, 5, 7)

In the first case, when the zone is 4 we're sure there's a collision since the center of the circle is __inside__ the box so we shall quickly return collision detected.

In the corner zones we'll find the closest corner to the circle and check whether it is inside the circle or not.

In the side zones we'll only check the distance by one coordinate : x distance between the centers of the objects for zones 3 and 5 and the y distance between the centers for zones 1 and 7. There's really nothing more to say, so code sample follows

{% highlight c++ %}
bool collisionDetected = false;
switch (zone) {
    // top and bottom side zones
    // check vertical distance between centers
    case 1:
    case 7:
        float distY = fabs(circle.y - box.y);
        if (distY <= (circle.radius + box.halfHeight))
            collisionDetected = true;
    break;

    // left and right side zones. check distance between centers
    // check horizontal distance between centers
    case 3:
    case 5:
        float distX = fabs(circle.x - box.x);
        if (distX <= (circle.radius + box.halfWidth))
            collisionDetected = true;
    break;

    // inside zone. collision for sure
    case 4:
        collisionDetected = true;
    break;

    // corner zone.
    // get the corner and check if inside the circle
    default:
        float cornerX = (zone == 0 || zone == 6) ? box.x - box.halfWidth : 
                                                   box.x + box.halfWidth;

        float cornerY = (zone == 0 || zone == 2) ? box.y - box.halfHeight : 
                                                   box.y + box.halfHeight;

        if (circle.isPointInside(cornerX, cornerY))
            collisionDetected = true;
    break;
}
{% endhighlight %}

One last example on which we can follow the above code flow:

![grid2](/public/images/grid2.png "Grid based collision detection")

From the image we can see the circle's center zone is 2, which is a corner zone. All we need to do is check whether the top-right box corner is inside the circle which is true in this case, therefor the box and circle collide.

All credits go to \_3yE\_. Thank you for your help !

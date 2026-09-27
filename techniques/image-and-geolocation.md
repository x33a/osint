# Image and geolocation

Techniques for working out where a photo was taken, what's in it, and where it came from.

- [Coordinates](#coordinates)
- [Reading text from above](#reading-text-from-above)
- [Country from a photo](#country-from-a-photo)
- [Reverse image search](#reverse-image-search)
- [Image metadata (EXIF)](#image-metadata-exif)
- [Tracing a photo to its original poster](#tracing-a-photo-to-its-original-poster)

---

## Coordinates

1. Paste `lat, lon` (with the comma) into Google Maps or openstreetmap.org to drop a pin.
2. Drag the Street View figure onto a blue-highlighted road to see the place at street level.
3. Use Street View's clock icon ("See more dates") or Google Earth Pro's historical imagery to see how the spot looked in the past.

Reading coordinates:

- Latitude (first number): positive = north of the equator.
- Longitude (second number): positive = east of the Prime Meridian.
- Five decimal places ≈ 1 m. Very long decimals usually mean the point was copied straight from a map pin, so the answer is *at* the pin, not nearby.

---

## Reading text from above

Questions like "what does it say here?" often mean text visible from the sky: rooftops, painted car parks, lettering mown into grass.

1. Switch Google Maps to **Satellite** and zoom in on the pin.
2. If it's blurry, or the pin covers the text, try **[Google Earth](https://earth.google.com)**: sharper imagery, a smaller marker, and you can rotate until the text reads left to right.
3. Try other providers — Bing Maps (Aerial) and Apple Maps use their own photos.
4. Try Google Earth Pro's **View → Historical Imagery** if the text may have changed.

Exact spelling and word choice usually matter for the answer.

---

## Country from a photo

1. **Script:** Arabic, Cyrillic, Thai, Hangul, Greek and so on narrow it immediately.
2. **Language-specific letters:** Polish ł ż, Czech ř, Hungarian ő, Turkish ğ ı, Ukrainian ї є.
3. **Local brands:** national dairy, drink and snack brands are strong clues. Search any unfamiliar name.
4. **Barcodes:** the first three digits of an EAN show where the code was registered:

   | Prefix | Country |
   | --- | --- |
   | 000–139 (UPC starting 0) | US / Canada |
   | 400–440 | Germany |
   | 460–469 | Russia |
   | 482 | Ukraine |
   | 500–509 | UK |
   | 590 | Poland |
   | 628 | Saudi Arabia |
   | 629 | UAE |
   | 880 | South Korea |

5. **Prices, currency symbols, plugs and bottle-deposit logos.**
6. **Outdoors:** driving side, road-line colours, number plates, bollards, street signs, vegetation.

Look for two or three clues that agree.

**Caution:** product origin, barcodes and label language show where goods were *made or registered*, not where the photo was taken. Tech products especially ship worldwide.

---

## Reverse image search

Each engine indexes different images, so try several.

| Engine | Best for |
| --- | --- |
| [Google Lens](https://lens.google.com) | Products, landmarks; accepts extra keywords with the image |
| [Yandex Images](https://yandex.com/images) | Exact visual matches of objects and faces of places |
| [Bing Visual Search](https://www.bing.com/visualsearch) | A third set of results |
| [TinEye](https://tineye.com) | Exact copies of a photo; sort by **oldest** to find the original post |

- Crop tight to the key object, and also search the full image.
- Add keywords alongside the image in Google Lens — including the challenge title or question text.
- Search for a larger, sharper copy when the image is too small to read a logo.

---

## Image metadata (EXIF)

Photos straight from a phone often carry EXIF metadata: GPS coordinates, camera model, date taken.

```bash
exiftool photo.jpg                 # everything
exiftool -gps:all -n photo.jpg     # just GPS, as decimal numbers
```

No terminal: [jimpl.com](https://jimpl.com) or [exif.tools](https://exif.tools).

Always download the **original file**, not a screenshot of the page. Screenshots, and most social media uploads, strip all metadata — if a PNG has only `ImageWidth`/`ImageHeight` and nothing else, it's a screenshot and EXIF won't help.

---

## Tracing a photo to its original poster

When the objects in a photo point to the wrong place and the file has no metadata, find who first posted it. Their profile or community usually gives the location.

1. Rule out the easy clues first (above), and note what they *can't* tell you.
2. Search the exact combination of items and era on Reddit and hobby forums (for example, specific product editions released in the same month).
3. Open the poster's history: other posts, comments and replies often reveal their country.
4. AI-assisted image search can surface the original thread. Verify its claims against the actual posts — keep solid evidence (the post itself, replies) separate from plausible-sounding context.

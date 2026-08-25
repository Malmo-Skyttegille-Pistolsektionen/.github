# Malmö Skyttegille Pistolsektionen

The pistol section of Malmö Skyttegille. This organization holds the software we
build for our own use on the range.

## Rotation Target

[**rotation_target**](https://github.com/Malmo-Skyttegille-Pistolsektionen/rotation_target)
runs timed shooting programs on a rotating target system. An ESP32-S3 board turns
the targets face-on and edge-on to a program and plays the spoken range commands
over an amplifier. A web app — served by the board itself over WiFi — starts and
stops programs, follows a run live, and manages the stored programs and audio.

📖 **[Documentation](https://malmo-skyttegille-pistolsektionen.github.io/rotation_target/)**

> [!WARNING]
> It moves steel on a live firing range, and it moves it on a timer. A target
> turns because a program said it was time — it has no sensor and no idea whether
> anyone is downrange. It is a convenience for running programs, **not a safety
> device**, and the range's own rules and range commands govern the line.
>
> Read the safety warning in the
> [project README](https://github.com/Malmo-Skyttegille-Pistolsektionen/rotation_target#readme)
> before installing or operating it.

## About this organization

Most of what we run is private and specific to the club, so only a couple of
repositories are visible here. Shared GitHub configuration — the org-wide
[Renovate](https://docs.renovatebot.com/) preset and the label set — lives in
[`.github`](https://github.com/Malmo-Skyttegille-Pistolsektionen/.github).

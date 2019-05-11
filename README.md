# Transcoding Service

This repository contains scripts that I use to setup automatic, persistent transcoding
of videos.

## Installation

Because this is just a personal side project, I haven't made much effort to make the installation story easy or pretty,
so there's some editing involved. But it's light-weight. Just edit the `start_transcoder_service` script and replace
the `/volume1/Media/Pending` path with with wherever your new videos will exist, and the `/volume1/Media/Staging` path
with wherever you want transcoded videos to show up.

Then run it by running `build` to create the Docker image locally, and then `start_transcoder_service` to get the
service running as a container named `transcoder`. To stop it later, just stop the `transcoder` Docker container.

Note that running multiple instances of the container in parallel isn't really likely to work.

## How it works

I preconfigured two rough video profiles -- one for DVD transcoding and one for Blu-ray.

The service lists files in `/input/Blu-ray` and `/input/DVD` and transcodes the first one that it can't find an existing
output file for. When done, it immediately looks for another one to transcode. If it can't find one that's in need of
transcoding, it sleeps for five minutes before looking again.

You shouldn't move any files out of `/output` before removing the corresponding file from `/input`, or the service may
start transcoding the input file again (since it may start running, and see the input file without a corresponding
output file and interpret it as pending).

The `transcode_one_bluray` function finds a pending Blu-ray file, determines the output file for it, and decides whether
the input file needs to be transcoded. If it does, it just calls `transcode_bluray` with the relevant file names, and
`transcode_bluray` calls `transcode_video` with bitrate, maxrate, and buffer size appropriate to my rough bluray
profile. The functions with `dvd` in the name do the same, just for DVD's.

If you want to use your own profile, you should be able to modify the `transcode_one_bluray` and `transcode_blu_ray` to
look in new directories and call `transcode_video` with new parameters pretty easily.

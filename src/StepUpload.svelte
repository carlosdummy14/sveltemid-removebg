<script lang="ts">
  import { Cloudinary } from "@cloudinary/url-gen";
  import { backgroundRemoval } from "@cloudinary/url-gen/actions/effect";

  import { ImageStatus } from "../types.d";
  import { imageStatus, modifiedImage, originalImage } from "./store";
  import Dropzone from "dropzone";
  import "dropzone/dist/dropzone.css";

  import { onMount } from "svelte";

  const cloudinary = new Cloudinary({
    cloud: {
      cloudName: "yourcloudname",
    },
    url: {
      secure: true,
    },
  });

  onMount(() => {
    const dropzone = new Dropzone("#dropzone", {
      uploadMultiple: false,
      acceptedFiles: ".jpg,.png,.webp",
      maxFiles: 1,
    });

    dropzone.on("sending", (file, xhr, formData) => {
      imageStatus.set(ImageStatus.UPLOADING);
      formData.append("upload_preset", "your upload preset");
      formData.append("timestamp", Date.now() / 1000);
      formData.append("api_key", "your api key");
    });

    dropzone.on("success", (file, response) => {
      const { public_id: publicId, secure_url: url } = response;

      const imageWithoutBackground = cloudinary
        .image(publicId)
        .effect(backgroundRemoval());

      console.log(imageWithoutBackground.toURL());

      imageStatus.set(ImageStatus.DONE);
      originalImage.set(url);
      modifiedImage.set(imageWithoutBackground.toURL());
    });

    dropzone.on("error", (file, response) => {
      console.log("Algo salio mal...");
    });
  });
</script>

<form
  id="dropzone"
  class="shadow-2xl border-dashed border-2 border-gray-300 rounded-lg aspect-video w-full flex justify-center items-center flex-col"
  action="https://api.cloudinary.com/v1_1/yourcloudname/image/upload"
>
  {#if $imageStatus === ImageStatus.READY}
    <button
      class="pointer-events-none bg-blue-600 rounded-full font-bold text-white text-xl px-6 py-4"
      >Upload files</button
    >
    <strong class="text-lg mt-4 text-gray-800">or drop a file</strong>
  {/if}

  {#if $imageStatus === ImageStatus.UPLOADING}
    <strong class="text-lg mt-4 text-gray-800">Uploading files...</strong>
  {/if}
</form>

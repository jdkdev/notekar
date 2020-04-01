<script>
  import { Field } from "$frontier-c";
  let name = "";
  let lastName = "Knight";
  let email = "";
  let password = "";
  let number = 2;
</script>

<section class="gx-span-full">
  <div>
    <h1>Buttons</h1>
    <button class="p-xxs">XXS Padding button</button>
    <button class="p-xs">XS Padding button</button>
    <button class="p-sm">SM Padding button</button>
    <button class="p">Normal button</button>
    <button class="p-md">MD Padding button</button>
    <button class="p-lm">LM Padding button</button>
    <button class="p-lg">LG Padding button</button>
    <button class="p-xl">Xl Padding button</button>
    <button class="p-xxl">XXl Padding button</button>
  </div>
  <Field name="name" value={name} />
  <Field name="email" type="email" value={email} />
  <Field name="password" type="password" value={password} />
  <Field label="Numberfield" type="number" name="number" value={number} />
  <Field name="last_name" value={lastName} />
</section>

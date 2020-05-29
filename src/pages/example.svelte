<script>
  import { Field } from "$frontier-c";
  let name = "";
  let lastName = "Knight";
  let email = "";
  let password = "";
  let number = 2;
</script>

<section class="">
  <h1>Buttons</h1>
  <div class="_m_">
    <button class="p__">XXS Padding button</button>
    <button class="p_">XS Padding button</button>
    <button class="p--">SM Padding button</button>
    <button class="p-">Normal button</button>
    <button class="p">MD Padding button</button>
    <button class="p+">LM Padding button</button>
    <button class="p++">LG Padding button</button>
    <button class="p*">Xl Padding button</button>
    <button class="p**">XXl Padding button</button>
  </div>
  <Field name="name" value={name} />
  <Field name="email" type="email" value={email} />
  <Field name="password" type="password" value={password} />
  <Field label="Numberfield" type="number" name="number" value={number} />
  <Field name="last_name" value={lastName} />
</section>

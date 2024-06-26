@Test
    public void testHasPrescriptionFillPropertyP_WhenPropertyPExists() {
        Property property = new Property();
        property.setName("P");
        PrescriptionFill prescriptionFill = new PrescriptionFill();
        prescriptionFill.setProperties(List.of(property));
        Item item = new Item();
        item.setPrescriptionFill(prescriptionFill);
        when(orderDto.getItems()).thenReturn(List.of(item));

        boolean result = orderService.hasPrescriptionFillPropertyP(orderDto);

        assertTrue(result);
    }

    @Test
    public void testHasPrescriptionFillPropertyP_WhenPropertyPDoesNotExist() {
        Property property = new Property();
        property.setName("Q");
        PrescriptionFill prescriptionFill = new PrescriptionFill();
        prescriptionFill.setProperties(List.of(property));
        Item item = new Item();
        item.setPrescriptionFill(prescriptionFill);
        when(orderDto.getItems()).thenReturn(List.of(item));

        boolean result = orderService.hasPrescriptionFillPropertyP(orderDto);

        assertFalse(result);
    }

    @Test
    public void testHasPrescriptionFillPropertyP_WhenPrescriptionFillIsNull() {
        Item item = new Item();
        item.setPrescriptionFill(null);
        when(orderDto.getItems()).thenReturn(List.of(item));

        boolean result = orderService.hasPrescriptionFillPropertyP(orderDto);

        assertFalse(result);
    }

    @Test
    public void testHasPrescriptionFillPropertyP_WhenPropertyListIsNull() {
        PrescriptionFill prescriptionFill = new PrescriptionFill();
        prescriptionFill.setProperties(null);
        Item item = new Item();
        item.setPrescriptionFill(prescriptionFill);
        when(orderDto.getItems()).thenReturn(List.of(item));

        boolean result = orderService.hasPrescriptionFillPropertyP(orderDto);

        assertFalse(result);
    }

    @Test
    public void testHasPrescriptionFillPropertyP_WhenItemsListIsNull() {
        when(orderDto.getItems()).thenReturn(null);

        boolean result = orderService.hasPrescriptionFillPropertyP(orderDto);

        assertFalse(result);
    }

    @Test
    public void testHasPrescriptionFillPropertyP_WhenItemsListIsEmpty() {
        when(orderDto.getItems()).thenReturn(Collections.emptyList());

        boolean result = orderService.hasPrescriptionFillPropertyP(orderDto);

        assertFalse(result);
    }

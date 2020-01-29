# Set Categories from Product Selection

```json
{
  type: 'measures/SET_CATEGORIES_FROM_PROPOSED_IMPROVEMENTS',
  response: {
    entities: {
      measures: {
        'energy_recovery_ventilation_(erv)_system': {
          category: 'HVAC',
          cost: 224000,
          name: 'Energy Recovery Ventilation (ERV) System',
          legacy_identifier: 'energy_recovery_ventilation_(erv)_system',
          eligibility_criteria: '1. Product must be certified by the Home Ventilating Institute (HVI).',
          lifespan: 15,
          triggers_shpo: false,
          product_type_question_sections: [
            {
              name: 'scope',
              questions: [
                {
                  identifier: 'cost',
                  title: 'Total Cost',
                  type: 'float'
                },
                {
                  identifier: 'quantity',
                  question_category: 'quantity',
                  title: 'Quantity',
                  type: 'integer'
                }
              ]
            },
            {
              name: 'details',
              questions: [
                {
                  identifier: 'manufacturer',
                  title: 'Manufacturer',
                  type: 'string'
                },
                {
                  identifier: 'model',
                  title: 'Model',
                  type: 'string'
                }
              ]
            },
            {
              name: 'verification',
              questions: [
                {
                  identifier: 'is_the_product_certified_by_the_home_ventilating_institute',
                  title: 'Is the product certified by the Home Ventilating Institute?',
                  type: 'boolean'
                }
              ]
            }
          ],
          max_days_to_close: 90,
          category_name: 'HVAC'
        }
      },
      categories: {
        HVAC: {
          name: 'HVAC',
          product_types: [
            'energy_recovery_ventilation_(erv)_system'
          ]
        }
      }
    },
    result: [
      'HVAC'
    ]
  }
}
```